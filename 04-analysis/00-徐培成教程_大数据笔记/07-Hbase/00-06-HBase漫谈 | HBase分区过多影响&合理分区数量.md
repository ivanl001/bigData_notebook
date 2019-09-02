https://blog.csdn.net/u011598442/article/details/90767150



# HBase漫谈 | HBase分区过多影响&合理分区数量

2019年06月04日 13:23:52 [大数据手稿笔记](https://me.csdn.net/u011598442) 阅读数 194

------

前段时间总结了一篇关于HBase由于分区过多导致集群宕机的文章，感兴趣的同学可以点击原文《[HBase案例 | 20000个分区导致HBase集群宕机事故处理](https://blog.csdn.net/u011598442/article/details/89468999)》阅读参考。本文重点参考HBase官网，从分区过多这个角度出发，进一步聊一聊HBase分区过多的影响以及单节点合理分区数量等。

## HBase 分区概念

接触过HBase的同学都知道，HBase每张表在底层存储上是由至少一个Region组成，Region实际上就是HBase表的分区。HBase新建一张表时默认Region即分区的数量为1，一般在生产环境中我们都会手动给Table提前做 “预分区”，使用合适的分区策略创建好一定数量的分区并使分区均匀分布在不同regionserver上。一个分区在达到一定大小时会自动Split，一分为二。

通常情况下，生产环境的每个regionserver节点上会有很多Region存在，我们一般比较关心每个节点上的Region数量，主要为了防止HBase分区过多影响到集群的稳定性。

## 切入主题：HBase分区过多有哪些影响？

分区过多会带来很多不好的影响，主要体现在以下几个方面。

***频繁刷写***

我们知道Region的一个列族对应一个MemStore，假设HBase表都有统一的1个列族配置，则每个Region只包含一个MemStore。通常HBase的一个MemStore默认大小为128 MB，见参数hbase.hregion.memstore.flush.size。当可用内存足够时，每个MemStore可以分配128 MB空间。当可用内存紧张时，假设每个Region写入压力相同，则理论上每个MemStore会平均分配可用内存空间。

因此，当节点Region过多时，每个MemStore分到的内存空间就会很小。这个时候，写入很小的数据量就会被强制Flush到磁盘，将会导致频繁刷写。频繁刷写磁盘，会对集群HBase与HDFS造成很大的压力，可能会导致不可预期的严重后果。

***压缩风暴***

因Region过多导致的频繁刷写，将在磁盘上产生非常多的HFile小文件，当小文件过多的时候HBase为了优化查询性能就会做Compaction操作，合并HFile减少文件数量。当小文件一直很多的时候，就会出现 “压缩风暴”。Compaction非常消耗系统io资源，还会降低数据写入的速度，严重的会影响正常业务的进行。

***MSLAB内存消耗较大***

MSLAB（MemStore-local allocation buffer）存在于每个MemStore中，主要是为了解决HBase内存碎片问题，默认会分配 2 MB 的空间用于缓存最新数据。如果Region数量过多，MSLAB总的空间占用就会比较大。比如当前节点有1000个包含1个列族的Region，MSLAB就会使用1.95GB的堆内存，即使没有数据写入也会消耗这么多内存。

***Master assign region时间较长***

HBase Region过多时Master分配Region的时间将会很长。特别体现在重启HBase时Region上线时间较长，严重的会达到小时级，造成业务长时间等待的后果。

***影响MapReduce并发数***

当使用MapReduce操作HBase时，通常Region数量就是MapReduce的任务数，Region数量过多会导致并发数过多，产生过多的任务。任务太多将会占用大量资源，当操作包含很多Region的大表时，占用过多资源会影响其他任务的执行。

## 具体计算HBase合理分区数量

关于每个regionserver节点分区数量大致合理的范围，HBase官网上也给出了定义：

> Generally less regions makes for a smoother running cluster (you can always manually split the big regions later (if necessary) to spread the data, or request load, over the cluster); 20-200 regions per RS is a reasonable range.

可见，通常情况下每个节点拥有20-200个Region是比较正常的。借鉴于20-200这个区间范围，我们接下来具体讨论。

实际上，每个regionserver的最大Region数量由总的MemStore内存大小决定。我们知道每个Region的每个列族对应一个MemStore，假设HBase表都有统一的1个列族配置，那么每个Region只包含一个MemStore。一个MemStore大小通常在128~256 MB，见参数hbase.hregion.memstore.flush.size。默认情况下，RegionServer会将自身堆内存的40%（见参数hbase.regionserver.global.memstore.size）供给节点上所有MemStore使用，如果所有MemStore的总大小达到该配置大小，新的更新将会被阻塞并且会强制刷写磁盘。因此，每个节点最理想的Region数量应该由以下公式计算（假设HBase表都有统一的列族配置）：

```java
((RS memory) * (total memstore fraction)) / ((memstore size)*(column families))
```

其中：

- RS memory：表示regionserver堆内存大小，即HBASE_HEAPSIZE。
- total memstore fraction：表示所有MemStore占HBASE_HEAPSIZE的比例，HBase0.98版本以后由hbase.regionserver.global.memstore.size参数控制，老版本由hbase.regionserver.global.memstore.upperLimit参数控制，默认值0.4。
- memstore size：即每个MemStore的大小，原生HBase中默认128M。
- column families：即表的列族数量，通常情况下只设置1个，最多不超过3个。

举个例子，假如一个集群中每个regionserver的堆内存是32GB，那么节点上最理想的Region数量应该是32768*0.4/128 ≈ 102，所以，当前环境中单节点理想情况下大概有102个Region。

这种最理想情况是假设每个Region上的填充率都一样，包括数据写入的频次、写入数据的大小，但实际上每个Region的负载各不相同，可能有的Region特别活跃负载特别高，有的Region则比较空闲。所以，通常我们认为23倍的理想Region数量也是比较合理的，针对上面举例来说，大概200300个Region算是合理的。

如果实际的Region数量比2~3倍的计算值还要多，就要实际观察Region的刷写、压缩情况了，Region越多则风险越大。经验告诉我们，如果单节点Region数量过千，集群可能存在较大风险。

## 总结

通过上述分析，我们大概知道在生产环境中，如果一个regionserver节点的Region数量在 20~200 我们认为是比较正常的，但是我们也要重点参考理论合理计算值。如果每个Region的负载比较均衡，分区数量在2~3倍的理论合理计算值通常认为也是比较正常的。

假设我们集群单节点Region数量比2~3倍计算值还要多，因为实际存在单节点分区数达到1000+/2000+的集群，遇到这种情况我们就要密切观察Region的刷写压缩情况了，主要从日志上分析，因为Region越多HBase集群的风险越大。经验告诉我们，如果单节点Region数量过千，集群可能存在较大风险。

------

关注我，一起进步。