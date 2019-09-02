### hbase架构解析,讲的还蛮好的，所以我这里就不总结了，看这篇文章吧：

- https://www.cnblogs.com/swordfall/p/8737328.html



每一个服务器上有一个HRegionServer服务器。

一个HRegionServer上可以有多个HRegion，也就是多张表。

一张表可能很大，所以有可能对应多个HRegion。但是每一个HRegion只存储一张表中内容。所以一张大表可以通过在不同的HRegionServer上创建HRegion存储在不同的服务器上。



每一个HRegion中有多个Store，每一个





* 在hbase中是有一个默认的命名空间的， 叫做“hbase”
* 该命名空间下有两个默认的表， 一个是namespace，用来存储hbase的所有命名空间，一个是meta，是元数据表，存储所有的分区信息，包括分区的服务器。这个meta表是至关重要的。

![IMAGE](../resources/82940140B832FB72AD6235EA0BE4E39D.jpg)

* 