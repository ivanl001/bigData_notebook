spark官网：[http://spark.apache.org/](http://spark.apache.org/)

spark官方文档：[http://spark.apache.org/docs/2.3.1/](http://spark.apache.org/docs/2.3.1/)



spark:

DAG：有向无环图



## 1, Spark的模块

Spark Core

- 提供内存计算和外部存储系统
- 为其他模块的计算提供引擎

Spark Streaming

- RDD
- 小批量而非实时计算

Spark SQL

- SchemaRDD

- 构建在Core模块之上，引入新的抽象，SchemaRDD，提供结构化和半结构化数据支持

- hive

SPark MLLib

- 比Mahout快9倍
- ALS(最小二乘法)-Alternating Least Square，也就是回归算法
- 

Spark Graphx

- 暂时不需要了解



## 2, RDD：spark的最基本的抽象

### 2.1, RDD包含五部分

```java
- RDD对数据集进行逻辑分区
- 顺便说一下， hdfs上文件是物理切割，也就是不管可不可切割，一律按照物理上的128M进行切分。合并后就是原文件。而MR则是逻辑切割，需要判断是否可切割，然后读取的时候，还要读取到一行末尾才行。
- MR的mapper的分区的个数，取决于Reducer的个数。(多说一句：mapper的个数则取决于文件的个数和文件的大小。)
- 而spark分区的个数，可以进行手动再分区。
- spark会把中间结果(也就是job的状态)存储到分布式内存中(分布式内存好像是用内存对齐技术，这个不太懂)

/**
 * 
 *  - A list of partitions
 *  - A function for computing each split
 *  - A list of dependencies on other RDDs
 *  - Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)
 *  - Optionally, a list of preferred locations to compute each split on (e.g. block locations for
 *    an HDFS file)
 * 
*/

RDD包含的五个部分：
----1, 分区列表
----2, 每个切片的计算函数
----3, 和其他RDD的依赖列表
----4, kv对RDD的分区器(这个是不一定有的，只有key-value类型的RDD才会有，用来shuffle的时候对RDD分区)
----5, 切片的首选位置(也就是hdfs文件的块位置，本地优先策略)

```



###  2.2, RDD变换

* RDD的api内容参考后面"01-徐培成-02-spark-transformation.md"的内容

*rdd是不可变的，类似val rdd3 = rdd2.map((_,1))这样的变换都是返回一个新的指针，然后进行下一步操作的*

*返回指向新rdd的指针，在rdd之间创建依赖关系。每个rdd都有计算函数和指向父RDD的指针。*

*RDD变换是lazy的，也就是在没有动作调用之前，其实都是不进行计算的，比如下面的代码：在collect运行之前，都不会打印*

```scala

//加载文本文件
val rdd1 = sc.textFile(args(0))

//压扁
val rdd2 = rdd1.flatMap(line => {
  //-------------------------------------------------
  //这里是有一行打印信息的，但是在最后那个collect执行前都不会打印这个信息，因为collect是触发的动作
  //-------------------------------------------------
  println("ivanl001" + line)
  line.split(" ")
})
    
//映射w => (w,1)
val rdd3 = rdd2.map((_,1))
val rdd4 = rdd3.reduceByKey(_ + _)


val r = rdd4.collect()
```