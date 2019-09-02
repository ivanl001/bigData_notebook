1, hive中如果小文件过多影响效率，因为一个文件会启动一个map，map过多必然不好

* 可以通过如下参数合并小文件

```mysql
-- 先处理输入的文件：合并输入的小文件，输入文件太多，会启动太多map，影响效率。可通过设置参数来合并小文件：
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
-- 然后处理输出等文件：合并输出小文件，以减少输出文件的大小，可通过如下参数设置：
set hive.merge.mapfiles=true; -- map only job结束时合并小文件
set hive.merge.mapredfiles=true; -- 合并reduce输出的小文件
-- set hive.merge.smallfiles.avgsize=256000000; -- 当输出文件平均大小小于该值，启动新job合并文件
-- sethive.merge.size.per.task=64000000; -- 合并之后的每个文件大小64M

set hive.exec.dynamic.partition.mode=nonstrict;
set hive.exec.max.dynamic.partitions=100000;
set hive.exec.max.dynamic.partitions.pernode=100000;
set hive.exec.max.created.files=655350;
set mapred.min.split.size.per.node=100000000; 
set mapred.min.split.size.per.rack=100000000;

insert overwrite  table ${tableName} partition(${partitions})
select * from ${tableName} where accounting_day=${accounting_day} ;
```

2，分桶的理由？

* 不太懂

```java
Hive为什么要分桶
对于每一个表（table）或者分区， Hive可以进一步组织成桶，也就是说桶是更为细粒度的数据范围划分。Hive也是针对某一列进行桶的组织。Hive采用对列值哈希，然后除以桶的个数求余的方式决定该条记录存放在哪个桶当中。

把表（或者分区）组织成桶（Bucket）有两个理由：

（1）获得更高的查询处理效率。桶为表加上了额外的结构，Hive 在处理有些查询时能利用这个结构。具体而言，连接两个在（包含连接列的）相同列上划分了桶的表，可以使用 Map 端连接 （Map-side join）高效的实现。比如JOIN操作。对于JOIN操作两个表有一个相同的列，如果对这两个表都进行了桶操作。那么将保存相同列值的桶进行JOIN操作就可以，可以大大较少JOIN的数据量。

（2）使取样（sampling）更高效。在处理大规模数据集时，在开发和修改查询的阶段，如果能在数据集的一小部分数据上试运行查询，会带来很多方便。

按我的理解，所谓Hive中的分桶，实际就是指的MapReduce中的分区。根据Reduce的数量，分成不同个数的文件。

注：1、order by 会对输入做全局排序，因此只有一个reducer，会导致当输入规模较大时，需要较长的计算时间。
2、sort by不是全局排序，其在数据进入reducer前完成排序。因此，如果用sort by进行排序，并且设置mapred.reduce.tasks>1，则sort by只保证每个reducer的输出有序，不保证全局有序。
3、distribute by(字段)根据指定的字段将数据分到不同的reducer，且分发算法是hash散列。
4、Cluster by(字段) 除了具有Distribute by的功能外，还会对该字段进行排序。
5、创建分桶表并不意味着load进数据也是分桶的，你必须先分好桶，然后再放到表中。


因此，如果分桶和sort字段是同一个时，此时，cluster by = distribute by + sort by

分桶表的作用：最大的作用是用来提高join操作的效率；但是两者的分桶数要相同或者成倍数。

为什么可以提高join操作的效率呢？因为按照MapReduce的分区算法，是Id的HashCode值模上ReduceTaskNumbers，所以一个ID会分到同一个桶中，这样合并就不用整个表遍历求笛卡尔积了，对应的桶合并就可以了。

https://blog.csdn.net/whdxjbw/article/details/82219022

https://blog.csdn.net/u010003835/article/details/80911215

https://blog.csdn.net/a280966503/article/details/79314013
```

