## 1, 什么是presto

presto是一个开源的分布式SQL查询引擎，主要用于秒级查询场景

`⚠️注意：虽然presto可以解析sql，但它不是一个标准的数据库。不是mysql，oracle的代替品，也不能用来处理在线事物（OLTP），它数据在线分析（OLAP）`



## 2, presto的架构

presto由一个Coordinator和多个worker组成

![image-20190731102915302](../../03-%E5%A4%A7%E6%95%B0%E6%8D%AE%E8%87%AA%E5%AD%A6v2.0/11-presto/assets/image-20190731102915302.png)



## 3, presto优缺点

基于内存，减少磁盘IO，计算速度快

支持丰富的数据源，这点比Impala还好

但是不能支持超过指定内存的数据计算，这样性能会降低，因为它是基于内存的

presto可以跨表查询





相对于Impala：

impala的性能稍微领先于presto， 但是presto在数据源支持上非常丰富。