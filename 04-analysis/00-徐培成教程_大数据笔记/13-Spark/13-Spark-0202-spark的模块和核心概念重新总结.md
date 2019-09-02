## 1, Spark的模块

Spark Core

* 提供内存计算和外部存储系统
* 为其他模块的计算提供引擎

Spark Streaming

* RDD
* 小批量而非实时计算

Spark SQL

* SchemaRDD

* 构建在Core模块之上，引入新的抽象，SchemaRDD，提供结构化和半结构化数据支持

* hive

SPark MLLib

* 比Mahout快9倍
* ALS(最小二乘法)-Alternating Least Square，也就是回归算法
* 

Spark Graphx

* 暂时不需要了解



## 2, RDD

* RDD对数据集进行逻辑分区
* 顺便说一下， hdfs上文件是物理切割，也就是不管可不可切割，一律按照物理上的128M进行切分。合并后就是原文件。而MR则是逻辑切割，需要判断是否可切割，然后读取的时候，还要读取到一行末尾才行。
* MR的mapper的分区的个数，取决于Reducer的个数。(多说一句：mapper的个数则取决于文件的个数和文件的大小。)
* 而spark分区的个数，可以进行手动再分区。
* spark会把中间结果(也就是job的状态)存储到分布式内存中(分布式内存好像是用内存对齐技术，这个不太懂)
* 





