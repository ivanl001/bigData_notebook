1.2版本架构：官方文档地址：

https://hbase.apache.org/1.2/book.html#_architecture



## 65. Catalog Tables

The catalog table `hbase:meta` exists as an HBase table and is filtered out of the HBase shell’s `list` command, but is in fact a table just like any other.



### 65.1. -ROOT-

`-ROOT-` 表在HBase 0.96.0中已经被移除了。

The `-ROOT-` table kept track of the location of the `.META` table (the previous name for the table now called `hbase:meta`) prior to HBase 0.96. The `-ROOT-` table structure was as follows:

Key

- .META. region key (`.META.,,1`)

Values

- `info:regioninfo` (serialized [HRegionInfo](http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/HRegionInfo.html) instance of `hbase:meta`)
- `info:server` (server:port of the RegionServer holding `hbase:meta`)
- `info:serverstartcode` (start-time of the RegionServer process holding `hbase:meta`)



### 65.2. hbase:meta

The `hbase:meta` table (previously called `.META.`) keeps a list of all regions in the system. The location of `hbase:meta` was previously tracked within the `-ROOT-` table, but is now stored in ZooKeeper.

The `hbase:meta` table structure is as follows:

Key

- Region key of the format (`[table],[region start key],[region id]`)

Values

- `info:regioninfo` (serialized [HRegionInfo](http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/HRegionInfo.html) instance for this region)
- `info:server` (server:port of the RegionServer containing this region)
- `info:serverstartcode` (start-time of the RegionServer process containing this region)

When a table is in the process of splitting, two other columns will be created, called `info:splitA` and `info:splitB`. These columns represent the two daughter regions. The values for these columns are also serialized HRegionInfo instances. After the region has been split, eventually this row will be deleted.

|      | Note on HRegionInfo                                          |
| ---- | ------------------------------------------------------------ |
|      | The empty key is used to denote table start and table end. A region with an empty start key is the first region in a table. If a region has both an empty start and an empty end key, it is the only region in the table |

In the (hopefully unlikely) event that programmatic processing of catalog metadata is required, see the [Writables](http://hbase.apache.org/devapidocs/org/apache/hadoop/hbase/util/Writables.html#getHRegionInfo(byte[])) utility.



### 65.3. Startup Sequencing

First, the location of `hbase:meta` is looked up in ZooKeeper. Next, `hbase:meta` is updated with server and startcode values.

For information on region-RegionServer assignment, see [Region-RegionServer Assignment](https://hbase.apache.org/1.2/book.html#regions.arch.assignment).





## 66. Client(重点)

The HBase client finds the RegionServers that are serving the particular row range of interest. It does this by querying the `hbase:meta` table. See [hbase:meta](https://hbase.apache.org/1.2/book.html#arch.catalog.meta) for details. After locating the required region(s), the client contacts the RegionServer serving that region, rather than going through the master, and issues the read or write request. This information is cached in the client so that subsequent requests need not go through the lookup process. Should a region be reassigned either by the master load balancer or because a RegionServer has died, the client will requery the catalog tables to determine the new location of the user region.

客户端会先去zk找到hbase:meta的服务器，然后去服务器上找到meta元数据表，这张表里存储了hbase的分区信息，包含每个分区的服务器地址。这些信息都会被客户端缓存下来，防止再次查找。

然后如果有读写请求，客户端会直接联系RegionServer而不需要联系master，就可以进行操作。

如果region被改写，客户端会重新查询新的分区信息。



See [Runtime Impact](https://hbase.apache.org/1.2/book.html#master.runtime) for more information about the impact of the Master on HBase Client communication.

Administrative functions are done via an instance of [Admin](http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/client/Admin.html)





## 70. Regions(重点)

Regions are the basic element of availability and distribution for tables, and are comprised of a Store per Column Family. The hierarchy of objects is as follows:

```mysql
Table                    (HBase table)
    Region               (Regions for the table)
        Store            (Store per ColumnFamily for each Region for the table)
            MemStore     (MemStore for each Store for each Region for the table)
            StoreFile    (StoreFiles for each Store for each Region for the table)
                Block    (Blocks within a StoreFile within a Store for each Region for the table)
```

For a description of what HBase files look like when written to HDFS, see [Browsing HDFS for HBase Objects](https://hbase.apache.org/1.2/book.html#trouble.namenode.hbase.objects).



### 70.4. Region Splits

Regions split when they reach a configured threshold. Below we treat the topic in short. For a longer exposition, see [Apache HBase Region Splitting and Merging](http://hortonworks.com/blog/apache-hbase-region-splitting-and-merging/) by our Enis Soztutar.

Splits run unaided on the RegionServer; i.e. the Master does not participate. The RegionServer splits a region, offlines the split region and then adds the daughter regions to `hbase:meta`, opens daughters on the parent’s hosting RegionServer and then reports the split to the Master. See [Managed Splitting](https://hbase.apache.org/1.2/book.html#disable.splitting) for how to manually manage splits (and for why you might do this).

#### 70.4.1. Custom Split Policies

You can override the default split policy using a custom [RegionSplitPolicy](http://hbase.apache.org/devapidocs/org/apache/hadoop/hbase/regionserver/RegionSplitPolicy.html)(HBase 0.94+). Typically a custom split policy should extend HBase’s default split policy: [IncreasingToUpperBoundRegionSplitPolicy](http://hbase.apache.org/devapidocs/org/apache/hadoop/hbase/regionserver/IncreasingToUpperBoundRegionSplitPolicy.html).

The policy can set globally through the HBase configuration or on a per-table basis.

Configuring the Split Policy Globally in *hbase-site.xml*

```
<property>
  <name>hbase.regionserver.region.split.policy</name>
  <value>org.apache.hadoop.hbase.regionserver.IncreasingToUpperBoundRegionSplitPolicy</value>
</property>
```

Configuring a Split Policy On a Table Using the Java API

```
HTableDescriptor tableDesc = new HTableDescriptor("test");
tableDesc.setValue(HTableDescriptor.SPLIT_POLICY, ConstantSizeRegionSplitPolicy.class.getName());
tableDesc.addFamily(new HColumnDescriptor(Bytes.toBytes("cf1")));
admin.createTable(tableDesc);
----
```

Configuring the Split Policy On a Table Using HBase Shell

```
hbase> create 'test', {METHOD => 'table_att', CONFIG => {'SPLIT_POLICY' => 'org.apache.hadoop.hbase.regionserver.ConstantSizeRegionSplitPolicy'}},
{NAME => 'cf1'}
```

The default split policy can be overwritten using a custom [RegionSplitPolicy(HBase 0.94+)](http://hbase.apache.org/devapidocs/org/apache/hadoop/hbase/regionserver/RegionSplitPolicy.html). Typically a custom split policy should extend HBase’s default split policy: [ConstantSizeRegionSplitPolicy](http://hbase.apache.org/devapidocs/org/apache/hadoop/hbase/regionserver/ConstantSizeRegionSplitPolicy.html).

The policy can be set globally through the HBaseConfiguration used or on a per table basis:

```
HTableDescriptor myHtd = ...;
myHtd.setValue(HTableDescriptor.SPLIT_POLICY, MyCustomSplitPolicy.class.getName());
```

|      | The `DisabledRegionSplitPolicy` policy blocks manual region splitting. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 70.5. Manual Region Splitting

It is possible to manually split your table, either at table creation (pre-splitting), or at a later time as an administrative action. You might choose to split your region for one or more of the following reasons. There may be other valid reasons, but the need to manually split your table might also point to problems with your schema design.

Reasons to Manually Split Your Table

- Your data is sorted by timeseries or another similar algorithm that sorts new data at the end of the table. This means that the Region Server holding the last region is always under load, and the other Region Servers are idle, or mostly idle. See also [Monotonically Increasing Row Keys/Timeseries Data](https://hbase.apache.org/1.2/book.html#timeseries).
- You have developed an unexpected hotspot in one region of your table. For instance, an application which tracks web searches might be inundated by a lot of searches for a celebrity in the event of news about that celebrity. See [perf.one.region](https://hbase.apache.org/1.2/book.html#perf.one.region) for more discussion about this particular scenario.
- After a big increase in the number of RegionServers in your cluster, to get the load spread out quickly.
- Before a bulk-load which is likely to cause unusual and uneven load across regions.

See [Managed Splitting](https://hbase.apache.org/1.2/book.html#disable.splitting) for a discussion about the dangers and possible benefits of managing splitting completely manually.

|      | The `DisabledRegionSplitPolicy` policy blocks manual region splitting. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

