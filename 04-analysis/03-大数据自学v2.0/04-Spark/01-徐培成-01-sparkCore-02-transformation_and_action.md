#  spark-RDD transformations.

* 官方地址：

http://spark.apache.org/docs/2.1.0/streaming-programming-guide.html#transformations-on-dstreams



## 0-DataStreams的变换函数-官方内容

* 更多内容参考书籍《[TUTORIALSPOINT]apache_spark_tutorial.pdf》
* tranformation返回的是RDD
* 而action返回的是值

RDD transformations returns pointer to new RDD and allows you to create dependencies between RDDs. Each RDD in dependency chain (String of Dependencies) has a function for calculating its data and has a pointer (dependency) to its parent RDD.

Spark is lazy, so nothing will be executed unless you call some transformation or action that will trigger job creation and execution. Look at the following snippet of the word- count example.

Therefore, RDD transformation is not a set of data but is a step in a program (might be the only step) telling Spark how to get data and what to do with it.

Given below is a list of RDD transformations.

| S. No | Transformations | Meaning                                    |
| ----- | --------------------------------------- | ------------------------------------------------------------ |
| 1 | **map相关** | **map相关** |
| 1.1  | **map(func)**                           | 就是map，映射函数，和flink的类似，一对一映射 |
| 1.2 | **mapPartitions(func)** | 可以在每个分区开始前和开始后进行操作 |
| 1.3 | **mapPartitionsWithIndex(func)** | 可以在每个分区开始前和开始后进行操作，而且带有索引index |
| 1.4  | **flatMap(func)**                       | 和map映射函数类似，只不过一个输入可以映射到0个或者多个输出，可以一对多映射，比如一行文字映射到一个切开文字的列表 |
| 1.5 | mapValue() | 不关心key，只处理value，底层调用map |
|       |                                                          |                                                              |
| 2     | **分区相关**                                             | **分区相关**                                                 |
| 2.1   | **repartition(numPartitions)**                           | 重分区：Reshuffle the data in the RDD randomly to create either more or fewer partitions and balance it across them. `This always shuffles all data over the network.` |
| 2.2   | **coalesce(numPartitions)**                              | 降低分区数，只能降低不能增加。  Decrease the number of partitions in the RDD to numPartitions. Useful for running operations more efficiently after filtering down a large dataset. |
| 2.3   | **repartitionAndSortWithinPartitions(partitioner)**      | Repartition the RDD according to the given partitioner and, within each resulting partition, sort records by their keys. This is more efficient than calling repartition and then sorting within each partition because it can push the sorting down into the shuffle machinery. |
|       |                                                          |                                                              |
| 3     | **key相关**                                              | **key相关**                                                  |
| 3.1   | **reduceByKey(func, [numTasks])**                        | (K, V)  =>  (K, V) ,                                            根据key值进行聚合操作. **Note:** 默认情况下，本地模式会有两个并发度, 在集群模式下，并发度根据： `spark.default.parallelism`决定. 你可以通过设置参数 `numTasks` 来决定任务数 |
| 3.2   | **aggregateByKey(zeroValue)(seqOp, combOp, [numTasks])** | When called on a dataset of (K, V) pairs, returns a dataset of (K, U) pairs where the values for each key are aggregated using the given combine functions and a neutral "zero" value. Allows an aggregated value type that is different from the input value type, while avoiding unnecessary allocations. Like in groupByKey, the number of reduce tasks is configurable through an optional second argument. |
| 3.3   | **sortByKey([ascending], [numTasks])**                   | When called on a dataset of (K, V) pairs where K implements Ordered, returns a dataset of (K, V) pairs sorted by keys in ascending or descending order, as specified in the Boolean ascending argument. |
| 3.4   | **groupByKey([numTasks])**                               | (K, V)  =>  (K, Iterable<V>)                          `Note: 如果分组是为了聚合，比如说求每个key对应的总和或者平均值, 建议用reduceByKey或者aggregateByKey，这样的话性能会更好` |
|       |                                                          |                                                              |
| 4     | **类似数据库相关**                                       | **类似数据库相关**                                           |
| 4.1   | **join(otherDataset, [numTasks])**                       | (K, V) + (K, W) => (K, (V, W))                               |
| 4.2   | **union(otherDataset)**                                  | 和mysql的union作用类似                                       |
| 4.3   | **intersection(otherDataset)** | Returns a new RDD that contains the intersection of elements in the source dataset and the argument. |
| 4.4   | **distinct([numTasks]))** | Returns a new dataset that contains the distinct elements of the source dataset. |
| 4.5 | **cogroup(otherDataset, [numTasks])** |(K, V) + (K, W) => (K, (Iterable<V>, Iterable<W>)).                                          When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (Iterable<V>, Iterable<W>)) tuples. This operation is also called group With.|
| 4.6 | **cartesian(otherDataset)** |When called on datasets of types T and U, returns a dataset of (T, U) pairs (all pairs of elements).|
|       |                                                          |                                                              |
| 5     | **单独其他**                                             | **单独其他**                                                 |
| 5.1   | **filter(func)**                                         | 过滤，只选择其中一部分数据                                   |
| 5.2   | **sample(withReplacement, fraction, seed)**              | Sample a fraction of the data, with or without replacement, using a given random number generator seed. |
| 5.3   | **pipe(command, [envVars])**                             | Pipe each partition of the RDD through a shell command, e.g. a Perl or bash script. RDD elements are written to the process's stdin and lines output to its stdout are returned as an RDD of strings. |
|  | 后面是看官网添加的 ||
|  | **countByValue**() |When called on a DStream of elements of type K, return a new DStream of (K, Long) pairs where the value of each key is its frequency in each RDD of the source DStream.|
|  | **transform**(*func*) |Return a new DStream by applying a RDD-to-RDD function to every RDD of the source DStream. This can be used to do arbitrary RDD operations on the DStream.|
|  | **updateStateByKey**(*func*) |Return a new "state" DStream where the state for each key is updated by applying the given function on the previous state of the key and the new values for the key. This can be used to maintain arbitrary state data for each key.|



## 1, map相关

### 1.1, map(func)

*map(func)和mapPartitions(func)的区别*

* map是一对一的映射，一个输入只能映射到一个输出上

  * map仅仅是对每个word进行计算，//对每个元素进行变换，应用变换函数
  
  * 而mapPartitions能够识别分区，在某个分区开始或者结束的时候执行某些代码，//对每个分区进行应用变换，输入的Iterator,返回新的迭代器，可以对分区进行函数处理。
    
    
    
    ```scala
    //映射w => (w,1)
    val rdd3 = rdd2.map(word =>{
      println("开始---：" + word)
      val res = (word, 1)
      println("结束---:" + word)
      res
    })
    
    //这段代码只能在每个单词前或者后添加执行的逻辑，没有办法区分具体的分区
    开始---：ivanl001_posfix
    (ivanl001_posfix,1)
    结束---:ivanl001_posfix
    开始---：is_posfix
    (is_posfix,1)
    结束---:is_posfix
    开始---：the_posfix
    (the_posfix,1)
    结束---:the_posfix
    开始---：king_posfix
    (king_posfix,1)
    结束---:king_posfix
    开始---：of_posfix
    (of_posfix,1)
    结束---:of_posfix
    开始---：world!_posfix
    (world!_posfix,1)
    结束---:world!_posfix
    ```





### 1.2, mapPartitions(*func*)

*然后我们可以先用mapPartitions进行分区区分操作，然后再用map映射处理后的数据就可以了*

如果需要进行第三方数据链接，可以使用mapPartitions，不然使用map会频繁的创建关闭连接，浪费资源的

```scala
package im.ivanl001.bigData.Spark.A02_Transformations.A01_tranformation_map

import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable.ArrayBuffer

//wordcount主程序
object A0102_tranformation_map_mapPartitions {

  def main(args: Array[String]): Unit = {


    //0, 创建Spark配置对象
    val conf = new SparkConf()

    //1, 集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程

    //2, 通过conf创建sc上下文对象
    val sc = new SparkContext(conf)

    //3, 加载文本文件，数字4代表最小分区数，最终分区等于或者大于4
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)

    //4, 压扁
    val rdd2 = rdd1.flatMap(line => {
      println("ivanl001" + line)
      line.split(" ")
    })

    //5, 在分区内进行相关处理
    val rdd2_1 = rdd2.mapPartitions(it => {

      val t = Thread.currentThread().getName
      println("partition start----" + t)
      val buf = ArrayBuffer[String]()

      for (str <- it) {
        println("在mapPartitions中：" + str+ "_posfix")
        buf += str+ "_posfix"
      }

      println("partition end------")
      buf.iterator
    })


    //6, 映射成kv对：w => (w,1)
    val rdd3 = rdd2_1.map(word =>{
      println("开始---：" + word)
      val res = (word, 1)
      println("结束---:" + word)
      res
    })

    //7, 聚合得出结果,reduceByKey依然是变换哈
    val rdd4 = rdd3.reduceByKey(_ + _)


    //8, 调用collect动作，取出rdd4中对数组进行遍历得到最终结果
    val r = rdd4.collect()
    r.foreach(println(_))


    /*val config = new SparkConf()
    config.setAppName("WordCountApp_scala")
    config.setMaster("local")//local是死的，本地模式就是local，不能乱改
    val sc = new SparkContext(config)
    val rdd01 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt")
    val rdd02 = rdd01.flatMap(line => line.split(" "))
    val rdd03 = rdd02.map(word => (word, 1))
    val rdd04 = rdd03.reduceByKey(_ + _)
    val result = rdd04.collect()
    result.foreach(println)*/
  }
}


//可以看到，下面 partition start----只打印一次，因为这里只有一个分区而已，可以在这里设置3个并发
//val rdd1 = sc.textFile("/Users/ivanl001/Desktop/bigData/input/zhang.txt", 3)

partition start----
ivanl001张丹峰 
在mapPartitions中：张丹峰_posfix
ivanl001哈哈哈
在mapPartitions中：哈哈哈_posfix
ivanl001嘿嘿嘿
在mapPartitions中：嘿嘿嘿_posfix
ivanl001厉害啊
在mapPartitions中：厉害啊_posfix
ivanl001h
在mapPartitions中：h_posfix
partition end------
开始---：张丹峰_posfix
结束---:张丹峰_posfix
开始---：哈哈哈_posfix
结束---:哈哈哈_posfix
开始---：嘿嘿嘿_posfix
结束---:嘿嘿嘿_posfix
开始---：厉害啊_posfix
结束---:厉害啊_posfix
开始---：h_posfix
结束---:h_posfix
```



### 1.3, mapPartitionsWithIndex(func)

*mapPartitionsWithIndex和mapPartitions类似,只不过会传递分区索引进来，如下*

```scala
package im.ivanl001.bigData.Spark.A02_Transformations.A01_tranformation_map

import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable.ArrayBuffer

//wordcount主程序
object A0103_tranformation_map_mapPartitionsWithIndex {

  def main(args: Array[String]): Unit = {


    //0, 创建Spark配置对象
    val conf = new SparkConf()

    //1, 集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程

    //2, 通过conf创建sc上下文对象
    val sc = new SparkContext(conf)

    //3, 加载文本文件，数字4代表最小分区数，最终分区等于或者大于4
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)

    //4, 压扁
    val rdd2 = rdd1.flatMap(line => {
      println("ivanl001" + line)
      line.split(" ")
    })

    //5, 在分区内进行相关处理, 这次使用mapPartitionsWithIndex可以直接得出分区序号
    val rdd2_1 = rdd2.mapPartitionsWithIndex((index,it) => {

      val t = Thread.currentThread().getName
      println("partition start----" + t + "分区索引是:" + index)
      val buf = ArrayBuffer[String]()

      for (str <- it) {
        println("在mapPartitions中：" + str+ "_posfix")

        buf += str+ "_posfix"
      }

      println("partition end------")
      buf.iterator
    })


    //6, 映射成kv对：w => (w,1)
    val rdd3 = rdd2_1.map(word =>{
      println("开始---：" + word)
      val res = (word, 1)
      println("结束---:" + word)
      res
    })

    //7, 聚合得出结果,reduceByKey依然是变换哈
    val rdd4 = rdd3.reduceByKey(_ + _)


    //8, 调用collect动作，取出rdd4中对数组进行遍历得到最终结果
    val r = rdd4.collect()
    r.foreach(println(_))



    val rdd5 = rdd3.map(_._2).reduce(_+_)
    println("总的单词个数是：" + rdd5)


    /*val config = new SparkConf()
    config.setAppName("WordCountApp_scala")
    config.setMaster("local")//local是死的，本地模式就是local，不能乱改
    val sc = new SparkContext(config)
    val rdd01 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt")
    val rdd02 = rdd01.flatMap(line => line.split(" "))
    val rdd03 = rdd02.map(word => (word, 1))
    val rdd04 = rdd03.reduceByKey(_ + _)
    val result = rdd04.collect()
    result.foreach(println)*/
  }
}
```



### 1.4, flatMap(*func*)

* flatMap可以一个输入映射到多个输出，和flink的flatMap类似
* 01中的map函数则只能一对一输出

```scala
package im.ivanl001.bigData.Spark.A02_Transformations.A01_tranformation_map

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 09:28
  * #description : 
  *
  **/
object A0104_tranformation_map_flatMap {

  def main(args: Array[String]): Unit = {

    //1, 创建Spark配置对象, 并通过配置对象获取上下文
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程
    //通过conf创建sc
    val sc = new SparkContext(conf)

    //2, 加载文本文件，数字4代表最小分区数，最终分区等于或者大于4
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)
    println(rdd1.count())

    //3, flatMap操作
    val rdd2 = rdd1.flatMap(line => {
      line.split(" ")
    })

    //4, 对比可以看到，rdd1的个数是1，rdd2的个数是6，也就是说flatmap可以从一个输入映射到多个输出
    println("个数是：" + rdd2.count())
  }
}
```



### 1.5, mapValue()

```scala
package im.ivanl001.spark.A02_Transformations.A01_tranformation_map

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-31 11:39
  * #description : mapValues,比如说分组之后想要对相同的key进行一定的操作，可以直接mapValue，拿出数组，然后进行处理后返回
  *
  **/
object A0105_tranformation_map_mapValues {

  def main(args: Array[String]): Unit = {

    //0, 组装所有配置参数
    val imConfig = Map(
      "spark.core"    -> "local[*]",
      "spark.appName" -> "SparkSQL"
    )

    //1, 根据配置参数，创建配置并创建sparkCofig, spark, sc, ssc等
    val sparkConfig = new SparkConf().setMaster(imConfig("spark.core")).setAppName(imConfig("spark.appName"))
    val spark = SparkSession.builder().config(sparkConfig).getOrCreate()
    val sc = spark.sparkContext
    val scc = new StreamingContext(sc, Seconds(10))


    //2, 执行变换
    val rdd01 = sc.parallelize(List((1, "ivanl001"),(2, "ivanl002"),(3, "ivanl003"),(4, "ivanl004"),(5, "ivanl005"),(6, "ivanl006"),(7, "ivanl007")))

    val rdd02 = rdd01.mapValues(value => "I am " + value)
    val rdd03 = rdd02.mapPartitionsWithIndex((index, iter) => {

      for (num <- iter){
        println("分区索引是：" + index + ", num:" + num)
      }

      iter
    })
    rdd03.collect()


    //3, 关闭资源等等
    scc.stop()//这个本身也没有开，所以可以不管
    sc.stop()
    spark.close()

  }
}
```







## 2, 分区相关



### 2.1, repartition(*numPartitions*)

```java
package im.ivanl001.bigData.Spark.A02_Transformations.A02_tranformation_partition

import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable.ArrayBuffer

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 09:43
  * #description : 重分区
  *
  **/
object A0201_tranformation_partition_repartition {

  def main(args: Array[String]): Unit = {
    //1, 创建Spark配置对象, 并通过配置对象获取上下文
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[2]")
    val sc = new SparkContext(conf)

    //2, 加载文本文件，数字4代表最小分区数，最终分区等于或者大于4
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)
    println(rdd1.count())

    //3, flatMap操作
    val rdd2 = rdd1.flatMap(line => {
      line.split(" ")
    })

    //4, 对比可以看到，rdd1的个数是1，rdd2的个数是6，也就是说flatmap可以从一个输入映射到多个输出
    println("个数是：" + rdd2.count())

    //5, 我们根据mapPartitionsWithIndex看一下原先分区
    val rdd3 = rdd2.mapPartitionsWithIndex((index,it) => {

      val t = Thread.currentThread().getName
      println("partition 重新分区前 start----" + t + "分区索引是:" + index)
      val buf = ArrayBuffer[String]()

      for (str <- it) {
        buf += str
      }

      println("partition 重新分区前 end------")
      buf.iterator
    })

    //6, 然后我们重新分区,分成两个区
    val rdd4  = rdd3.repartition(2)

    //7, 我们根据mapPartitionsWithIndex看一下原先分区
    val rdd5 = rdd4.mapPartitionsWithIndex((index,it) => {

      val t = Thread.currentThread().getName
      println("partition 重新分区后 start----" + t + "分区索引是:" + index)
      val buf = ArrayBuffer[String]()

      for (str <- it) {
        buf += str
      }

      println("partition 重新分区后 end------")
      buf.iterator
    })

    //8, 输出结果
    rdd5.collect()
    rdd5.foreach(println)
  }
}


//结果打印如下：可以看到，原先是有五个分区，两个线程，现在变成两个分区，两个线程
1
个数是：6
partition 重新分区前 start----Executor task launch worker-1分区索引是:1
partition 重新分区前 start----Executor task launch worker-0分区索引是:0
partition 重新分区前 end------
partition 重新分区前 end------
partition 重新分区前 start----Executor task launch worker-1分区索引是:2
partition 重新分区前 end------
partition 重新分区前 start----Executor task launch worker-1分区索引是:3
partition 重新分区前 end------
partition 重新分区前 start----Executor task launch worker-1分区索引是:4
partition 重新分区前 end------
partition 重新分区后 start----Executor task launch worker-0分区索引是:0
partition 重新分区后 start----Executor task launch worker-1分区索引是:1
partition 重新分区后 end------
partition 重新分区后 end------
partition 重新分区后 start----Executor task launch worker-0分区索引是:0
partition 重新分区后 start----Executor task launch worker-1分区索引是:1
partition 重新分区后 end------
partition 重新分区后 end------
is
ivanl001
the
of
king
world!
```



### 2.2, coalesce(numPartitions)

*coalesce是减少分区，后者是重新分区，可以减少分区，也能增加分区*

* 其实上面说的不对
* coaleasce可以增加分区，也可以减少分区，只不过增加分区的时候需要打开shuffle才行
* repartition本身底层就是调用coaleasce

```scala
package im.ivanl001.bigData.Spark.A02_Transformations.A02_tranformation_partition

import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0202_tranformation_partition_coalesce {

  def main(args: Array[String]): Unit = {

    //0, 创建Spark配置对象
    val conf = new SparkConf()

    //1, 集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程

    //2, 通过conf创建sc上下文对象
    val sc = new SparkContext(conf)

    //3, 加载文本文件
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)//数字代表最小分区分区
    println("rdd1 partition:" + rdd1.partitions.length)

    //这个变换可以减少分区，返回一个rdd，通过新的rdd进行相关操作即可
    //但是要注意：这里只能减少分区，不能增加，如果需要增加用repartition
    val rdd01 = rdd1.coalesce(3)

    //4, 压扁
    val rdd2 = rdd01.flatMap(_.split(" "))
    println("rdd2 partition:" + rdd01.partitions.length)

    //通过repartition可以增加分区，其实用这个也能减少分区
    val rdd02 = rdd2.repartition(7)

    //5,映射
    val rdd3 = rdd02.map((_,1))
    println("rdd3 partition:" + rdd3.partitions.length)
  }
}


//打印结果如下
rdd1 partition:5
rdd2 partition:3
rdd3 partition:7
```







### 2.3, repartitionAndSortWithinPartitions(partitioner)

*重新分区并在分区内进行排序*

```scala

```



## 3, key相关

### 3.1, reduceByKey(*func*, [*numTasks*])

```scala
package im.ivanl001.bigData.Spark.A02_Transformations.A03_transformation_key

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 10:31
  * #description : 
  *
  **/
object A0301_transformation_key_reduceByKey {

  def main(args: Array[String]): Unit = {

    //0, 创建Spark配置对象,通过conf创建sc上下文对象
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程
    val sc = new SparkContext(conf)

    //1, 加载文本文件，数字4代表最小分区数，最终分区等于或者大于4
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)

    //2, 压扁
    val rdd2 = rdd1.flatMap(line => {
      println("ivanl001" + line)
      line.split(" ")
    })

    //3, 映射成kv对：w => (w,1)
    val rdd3 = rdd2.map(word => (word, 1))

    //4, 根据key值聚合得出结果,reduceByKey依然是变换哈
    val rdd4 = rdd3.reduceByKey(_ + _)

    //5, 调用collect动作，取出rdd4中对数组进行遍历得到最终结果
    val r = rdd4.collect()
    r.foreach(println(_))
  }
}


//打印结果如下；
ivanl001ivanl001 is the king of world!
(is,1)
(ivanl001,1)
(king,1)
(the,1)
(of,1)
(world!,1)
```



### 3.2, aggregateByKey(zeroValue)(seqOp, combOp, [numTasks])

```
具体看面试复习里面的spark部分内容
```





### 3.3, sortByKey([ascending], [numTasks])

*很明显，就是排序, 代码如下:*

```scala
package im.ivanl001.bigData.Spark.A02_Transformations

import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0296_WordCountApp_sortByKey {

  def main(args: Array[String]): Unit = {

    //1, 创建配置对象，并根据配置对象创建上下文
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)

    //2, 读取文件
    /*
    class01 ivanl001 male 10
    class02 ivanl003 male 20
    class03 ivanl333 female 30
    class01 ivanlddd male 13
    class03 ivanleee femal 23
    class01 iiiiiiii male 343
     */
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/students.txt",1)

    //3, 映射成k-v类型的rdd
    val rdd2 = rdd1.map(line => {
      val key = line.split(" ")(0)
      (key, line)
    })

    // 这里是有排序的, RDD[(K, V)]
    val rdd3 = rdd2.sortByKey()
    rdd3.foreach(e => {
      val key = e._1
      val value = e._2

      println("key值是:" + key + "-------"  + "value值是:" + value)
    })
  }
}

// 打印结果如下
key值是:class01-------value值是:class01 ivanl001 male 10
key值是:class01-------value值是:class01 ivanlddd male 13
key值是:class01-------value值是:class01 iiiiiiii male 343
key值是:class02-------value值是:class02 ivanl003 male 20
key值是:class03-------value值是:class03 ivanl333 female 30
key值是:class03-------value值是:class03 ivanleee female 23
```





### 3.4, groupByKey([numTasks])

*根据key分组,比较好理解，看代码*

> (K, V) => (K, Iterable<V>)

```scala
package im.ivanl001.bigData.Spark.A02_Transformations

import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0297_WordCountApp_groupByKey {

  def main(args: Array[String]): Unit = {

    //1, 创建配置对象，并根据配置对象创建上下文
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)

    //2, 读取文件
    /*
		class01 ivanl001 male 10
		class02 ivanl003 male 20
		class03 ivanl333 female 30
		class01 ivanlddd male 13
		class03 ivanleee femal 23
		class01 iiiiiiii male 343
     */
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/students.txt",1)

    //3, 映射成k-v类型的rdd
    val rdd2 = rdd1.map(line => {
      val key = line.split(" ")(0)
      (key, line)
    })

    //4, 根据key值进行分组
    //因为groupByKey把结果转换成(K, V) => (K, Iterable)这种，所以按照如下方式进行打印
    val rdd3 = rdd2.groupByKey()

    //5, 打印分组结果
    rdd3.foreach(e => {
      val key = e._1
      val iterable = e._2
      for (res <- iterable) {
        println("key值是:" + key + "-------"  + "value值是:" + res)
      }
      println("=====================")
    })
  }
}

//打印结果如下：
key值是:class02-------value值是:class02 ivanl003 male 20
=====================
key值是:class03-------value值是:class03 ivanl333 female 30
key值是:class03-------value值是:class03 ivanleee female 23
=====================
key值是:class01-------value值是:class01 ivanl001 male 10
key值是:class01-------value值是:class01 ivanlddd male 13
key值是:class01-------value值是:class01 iiiiiiii male 343
=====================
```



## 4, 类似数据库相关

### 4.1, join(otherDataset, [numTasks])

*上面05中已经讲过union，union就是把两个rdd的内容合并到一个rdd中去，是纵向的，join不一样，join是横向的*
*方式大概是: (K, V) and (K, W)  =>  (K, (V, W)),看起来和groupByKey有点像对吧，但是前者是针对多张表进行的join，而group是在单张表中进行分组的哈*

```scala
package im.ivanl001.bigData.Spark.A02_Transformations.A04_transformation_mysql

import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0401_transformation_mysql_join {

  def main(args: Array[String]): Unit = {

    //1,创建配置对象，根据配置对象创建上下文
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)

    //2, 读取文件
    /*
    customer.txt
    001 ivanl001
    001 ivanl006
    002 ivanl002
    003 ivanl003
    004 ivanl004
    orders.txt
    001 apple 10
    001 pig 200
    003 food 333
    004 cup 342
    004 clothes 3432
     */
    val customerRdd = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/customer.txt",1)
    val ordersRdd = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/orders.txt",1)

    //3, 根据文件进行切分
    val customerRdd2 = customerRdd.map(line => (line.split(" ")(0),line))
    val orderRdd2 = ordersRdd.map(line => (line.split(" ")(0),line))

    //4, 进行join操作
    val rdd = customerRdd2.join(orderRdd2)

    //5.1, 第一种打印方式
    /*rdd.foreach(e => {
      val key = e._1
      val value = e._2
      println(key + ":::::" + value)
    })*/

    //5.2, 第二种打印方式
    rdd.collect().foreach(println)
  }
}

// 打印结果如下
(003,(003 ivanl003,003 food 333))
(004,(004 ivanl004,004 cup 342))
(004,(004 ivanl004,004 clothes 3432))
(001,(001 ivanl001,001 apple 10))
(001,(001 ivanl001,001 pig 200))
(001,(001 ivanl006,001 apple 10))
(001,(001 ivanl006,001 pig 200))

```

### 4.2, **union**(*otherStream*)

*意思就是把两个rdd的内容合并到一个rdd中去*

```scala
package im.ivanl001.bigData.Spark.A02_Transformations
import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0204_WordCountApp_Union {

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)

    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/bigData/input/log.txt",2)

    //所有error
    //这里是单独的带有error的行
    val errorRDD = rdd1.filter(_.toLowerCase.contains("error"))
    errorRDD.foreach(line => {
      println(line)
      println("-----")
    })

    //所有warn行
    //这里是单独的带有warn的行
    val warnRDD = rdd1.filter(_.toLowerCase.contains("warn"))
    warnRDD.foreach(line => {
      println(line)
      println("++++")
    })

    //下面可以通过union的方式把所有的行放在一起然后放在一个RDD中
    val allRDD = errorRDD.union(warnRDD)
    println("==================")
    allRDD.collect().foreach(println)
  }
}
```

### 4.3, intersection(otherDataset)

*返回两个rdd中有交集的部分*

```scala
package im.ivanl001.bigData.Spark.A02_Transformations
import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0205_WordCountApp_Intersection {

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/bigData/input/log.txt",4)
    //所有error
    val errorRDD = rdd1.filter(_.toLowerCase.contains("error"))

    //所有warn行
    val warnRDD = rdd1.filter(_.toLowerCase.contains("warn"))

    //这里可以通过intersection返回两个rdd中相同的部分，也就是有交集的地方
    val intersecRDD = errorRDD.intersection(warnRDD)
    intersecRDD.collect().foreach(println)
  }
}
```



### 4.4, distinct([numTasks]))

*这个可以取出某个rdd中重复的内容*

* 会有shuffle

```scala
package im.ivanl001.bigData.Spark.A02_Transformations.A04_transformation_mysql

import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0404_transformation_mysql_distinct {

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)

    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/customer.txt")
    //distince去重是对这个rdd去重，而不是对key去重
    val rdd2 = rdd1.map(line => (line.split(" ")(0), line)).distinct()
    rdd2.foreach(println)


    /*val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/log.txt",4)
    //所有error
    //val errorRDD = rdd1.filter(_.toLowerCase.contains("error"))

    //所有warn行
    val warnRDD = rdd1.filter(_.toLowerCase.contains("warn"))
    //这里是可以打印出相同的那些行的
    warnRDD.foreach(println)

    println("--------------------------------------")

    //这里可以通过distinct去掉某个rdd中重复的部分
    //下面经过去重之后就不会打印出相同的行了，内容相同的行只能打印出一行哈
    val distinctedWarnRDD = warnRDD.distinct()
    distinctedWarnRDD.foreach(println)*/
  }
}
```





### 4.5, cogroup(otherDataset, [numTasks])

*(K, V) and (K, W)  =>  (K, (Iterable<V>, Iterable<W>))*

```scala
/*
意思大概知道了，这里就不再写练习了
比如说有很多学校，一张表记录每个学校的老师，另外一张表记录每个学校的学生，想要把每个学校的老师学生放在一起就可以用这个cogroup，因为一个学校会对应多个老师,一个学校也会对应多个学生
*/

package im.ivanl001.bigData.Spark.A02_Transformations

import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0210_WordCountApp_cogroup {

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)

    //表中的数据并不符合实际情况，不过这里就是为了演示一下协分组的，没必要深究
    val customerRdd = sc.textFile("/Users/ivanl001/Desktop/bigData/input/customer01.txt",1)
    val ordersRdd = sc.textFile("/Users/ivanl001/Desktop/bigData/input/orders01.txt",1)

    val customerRdd2 = customerRdd.map(line => (line.split(" ")(0),line))
    val orderRdd2 = ordersRdd.map(line => (line.split(" ")(0),line))

    val rdd = customerRdd2.cogroup(orderRdd2)

    rdd.foreach(e => {

      println("key:" + e._1)

      val val01 = e._2._1
      val val02 = e._2._2

      for (value <- val01) {
        print(value + "  ")
      }
      println()
      for (value <- val02) {
        print(value + "  ")
      }
      println()
    })
  }
}
//结果是:可以重点看下key=1的时候，这个时候不论是顾客还是商品都是对应多个，这个也就是故意做的数据，为了演示一下协分组而已，不必在意逻辑性
/*
key:2
2 ivanl002  
2 ae  
key:3
3 ivanl003  
3 西红柿  3 苹果  
key:1
1 ivanl001  1 ivanl0001  1 ivanl00001  
1 eos5dMark4  1 iphone  1 finalCut  
*/
```

### 4.6, cartesian(otherDataset)

*笛卡尔积，就是把两张表的内容进行笛卡尔积获取，应该用的也不多*

```scala
package im.ivanl001.bigData.Spark.A02_Transformations
import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0211_WordCountApp_cartesian {

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)

    //这里用暴力破解密码的案例演示一下笛卡尔积，只知道几个用户名和几个密码，不知道怎么对应，就用笛卡尔积试试
    val usernameRdd = sc.textFile("/Users/ivanl001/Desktop/bigData/input/username.txt",1)
    val passwordRdd = sc.textFile("/Users/ivanl001/Desktop/bigData/input/password.txt",1)

    val result = usernameRdd.cartesian(passwordRdd)
    result.foreach(println(_))
  }
}
//一共三个用户名，三个密码，结果会是九个，如下:
/*
(ivanl001,13579)
(ivanl001,24680)
(ivanl001,12345)
(ivanl002,13579)
(ivanl002,24680)
(ivanl002,12345)
(ivanl003,13579)
(ivanl003,24680)
(ivanl003,12345)
*/
/*
val usernameRdd = sc.parallelize(Array("ivanl001", "ivanl002", "ivanl003"))
val passwordRdd = sc.parallelize(Array("123", "456", "789"))
创建rdd的时候，也可以通过上面的方式
*/
```



## 5, 单独其他

### 5.1, filter(*func*)

```scala
def main(args: Array[String]): Unit = {

  //1, 创建Spark配置对象, 并通过配置对象获取上下文
  val conf = new SparkConf()
  conf.setAppName("WordCountScala")
  conf.setMaster("local[2]")
  val sc = new SparkContext(conf)

  //2, 加载文本文件
  val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)//数字代表最小分区数, 最终分区数等于或者大于这个数字
  println(rdd1.count())

  //3, flatMap操作
  val rdd2 = rdd1.flatMap(line => {
    line.split(" ")
  })

  //4, 对比可以看到，rdd1的个数是1，rdd2的个数是6，也就是说flatmap可以从一个输入映射到多个输出
  println("个数是：" + rdd2.count())

  //5, 进行过滤操作
  val rdd3 = rdd2.filter(word => {
    if (word.equals("ivanl001")) {
      true
    } else {
      false
    }
  })

  //6, 打印过滤结果,可以看到，结果里面只剩下ivanl001了
  rdd3.collect()
  rdd3.foreach(println)
}
```



























### 5.2, sample(withReplacement, fraction, seed)

*采样后返回新的rdd，后续使用新的rdd可以减少数据倾斜问题*

```scala
package im.ivanl001.bigData.Spark.A02_Transformations
import org.apache.spark.{SparkConf, SparkContext}
import scala.collection.mutable.ArrayBuffer

//wordcount主程序
object A0203_WordCountApp_Sample {

  def main(args: Array[String]): Unit = {
    
    //0, 创建Spark配置对象
    val conf = new SparkConf()

    //1, 集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程

    //2, 通过conf创建sc上下文对象
    val sc = new SparkContext(conf)

    //3, 加载文本文件
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/bigData/input/zhang.txt", 4)//数字代表最小分区数, 最终分区数等于或者大于这个数字

    //4, 压扁
    val rdd2 = rdd1.flatMap(line => {
//      println("ivanl001" + line)
      line.split(" ")
    })

    //5,教程里面没具体讲，大概就是这里采样一下，给出新的rdd，就差不多可以直接用于后续操作了，因为这个RDD采样后已经处理过了
    val sampledRdd = rdd2.sample(false, 0.5)
    sampledRdd.foreach(println(_))
  }
}
```



### 5.3, pipe(command, [envVars])

*这个可以执行shell脚本命令，把结果包装成rdd返回*

```scala
package im.ivanl001.bigData.Spark.A02_Transformations.A05_transformation_single

import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0503_transformation_single_pipe {

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)

    val rdd01 = sc.parallelize("/Users/ivanl001/")
    //val rdd02 = rdd01.pipe("ls")
    val rdd02 = rdd01.pipe("ls /Users/ivanl001/Desktop/")
    rdd02.foreach(println(_))

    println("===============================================")

    val rdd03 = rdd01.pipe("ls /Users/ivanl001/")
    rdd03.foreach(println(_))

    println("===============================================")

    val rdd04 = rdd01.pipe("cp /Users/ivanl001/Desktop/00-bigData/abc.jpg /Users/ivanl001/Desktop")
    rdd04.foreach(println(_))
  }
}
```



## 6, 看官网添加的

### 6.1, countByValue()

> When called on a DStream of elements of type K, return a new DStream of (K, Long) pairs where the value of each key is its frequency in each RDD of the source DStream.
>
> 对于单个元素的DStream，可以直接计算出元素的个数
>
> 可以看到，用这个可以直接计算wordcount

```scala
package im.ivanl001.bigData.Spark.A02_Transformations.A06_transformation_added

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 10:31
  * #description : 这种用法一般比较少感觉。
  *
  **/
object A9999_transformation_countByValue {

  //这个等下再说
  def main(args: Array[String]): Unit = {

    //1, 创建配置对象，并根据配置对象创建上下文
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)

    //2, 读取文件
    /*
    class01 ivanl001 male 10
    class02 ivanl003 male 20
    class03 ivanl333 female 30
    class01 ivanlddd male 13
    class03 ivanleee female 23
    class01 iiiiiiii male 343
     */
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt",1)

    //3, 映射成k-v类型的rdd
    val rdd2 = rdd1.flatMap(_.split(" "))

    //4, 根据value值计算个数，如果key不一样，是分到两组里计算个数的哦
    val rdd3 = rdd2.countByValue()
    rdd3.foreach(e => {
      val key = e._1
      val value = e._2

      println("key值是:" + key + "-------"  + "个数是:" + value)
    })
  }
}

// 打印结果如下：
key值是:is-------个数是:1
key值是:world!-------个数是:1
key值是:ivanl001-------个数是:1
key值是:king-------个数是:1
key值是:of-------个数是:1
key值是:the-------个数是:1
```



### 6.2, **transform**(*func*)



### 6.3, **updateStateByKey**(*func*)



# spark-RDD action

## 1-DataStreams的动作函数-官方内容

The following table gives a list of Actions, which return values.

| S.No |                                               | Action & Meaning                                             |
| ---- | --------------------------------------------- | ------------------------------------------------------------ |
| 1    | 取全部结果                                    |                                                              |
| 1.1  | **collect()**                                 | Returns all the elements of the dataset as an array at the driver program. This is usually useful after a filter or other operation that returns a sufficiently small subset of the data. |
| 1.2  | **foreach(func)**                             | Runs a function func on each element of the dataset. This is usually, done for side effects such as updating an Accumulator or interacting with external storage systems.  Note: modifying variables other than Accumulators outside of the foreach() may result in undefined behavior. See Understanding closures for more details. |
|      |                                               |                                                              |
| 2    | 取部分结果                                    |                                                              |
| 2.1  | **first()**                                   | Returns the first element of the dataset (similar to take (1)). |
| 2.2  | **take(n)**                                   | Returns an array with the first n elements of the dataset.   |
| 2.3  | **takeOrdered(n, [ordering])**                | Returns the first n elements of the RDD using either their natural order or a custom comparator. |
| 2.4  | **takeSample(withReplacement,num, [seed])**   | Returns an array with a random sample of num elements of the dataset, with or without replacement, optionally pre-specifying a random number generator seed. |
|      |                                               |                                                              |
| 3    | 取结果聚合                                    |                                                              |
| 3.1  | **reduce(func)**                              | Aggregate the elements of the dataset using a function func (which takes two arguments and returns one). The function should be commutative and associative so that it can be computed correctly in parallel. |
| 3.2  | **count()**                                   | Returns the number of elements in the dataset.               |
| 3.3  | **countByKey()**                              | Only available on RDDs of type (K, V). Returns a hashmap of (K, Int) pairs with the count of each key. |
|      |                                               |                                                              |
| 4    | 保存结果                                      |                                                              |
| 4.1  | **saveAsTextFile(path)**                      | Writes the elements of the dataset as a text file (or set of text files) in a given directory in the local filesystem, HDFS or any other Hadoop-supported file system. Spark calls toString on each element to convert it to a line of text  in the file. |
| 4.2  | **saveAsSequenceFile(path) (Java and Scala)** | Writes the elements of the dataset as a Hadoop SequenceFile in a given path in the local filesystem, HDFS or any other Hadoop-supported file system. This is available on RDDs of key-value pairs that implement Hadoop's Writable interface. In Scala, it is also available on types that are implicitly convertible to Writable (Spark includes conversions for basic types like Int, Double, String, etc). |
| 4.3  | **saveAsObjectFile(path) (Java and Scala)**   | Writes the elements of the dataset in a simple format using Java serialization, which can then be loaded using SparkContext.objectFile(). |
|      |                                               |                                                              |



## 1, 取全部结果

### 1.1, collect()

```java
package im.ivanl001.bigData.Spark.A03_Action.A01_action_result_all

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 16:40
  * #description : 
  *
  **/
object A0101_action_all_collect {

  def main(args: Array[String]): Unit = {

    //1, 创建Spark配置对象并根据对象创建上下文
    val conf = new SparkConf()
      //集群模式下下面两行不要
      conf.setAppName("WordCountScala")
      //设置master属性
      //conf.setMaster("spark://master:7077")
      conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程
      //通过conf创建sc
      val sc = new SparkContext(conf)

      //2, 加载上下文
      //加载文本文件，数字4代表最小分区数，最终分区等于或者大于4
      //val rdd1 = sc.textFile(args(0))
      val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)
      /*val rdd1_1 = rdd1.map(line => line.split(" ").length)
    val count01 = rdd1_1.reduce(_ + _)
    println("个数是:" + count01)*/

      //3, 一对多映射
      val rdd2 = rdd1.flatMap(line => {
        println("ivanl001" + line)
          line.split(" ")
      })

      println("个数是：" + rdd2.count())

      //4, 映射w => (w,1)
      val rdd3 = rdd2.map(word => (word, 1))

      //5, 根据key聚合
      val rdd4 = rdd3.reduceByKey(_ + _)

      //6, collect
      val r = rdd4.collect()
      r.foreach(println)
  }
}
```



### 1.2, foreach(func)

```java
package im.ivanl001.bigData.Spark.A03_Action.A01_action_result_all

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 16:40
  * #description : 
  *
  **/
object A0102_action_all_foreach {
  def main(args: Array[String]): Unit = {

    //1, 创建Spark配置对象并根据对象创建上下文
    val conf = new SparkConf()
    //集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程
    //通过conf创建sc
    val sc = new SparkContext(conf)

    //2, 加载上下文
    //加载文本文件，数字4代表最小分区数，最终分区等于或者大于4
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)
    /*val rdd1_1 = rdd1.map(line => line.split(" ").length)
    val count01 = rdd1_1.reduce(_ + _)
    println("个数是:" + count01)*/

    //3, 一对多映射
    val rdd2 = rdd1.flatMap(line => {
      println("ivanl001" + line)
      line.split(" ")
    })

    println("个数是：" + rdd2.count())

    //4, 映射w => (w,1)
    val rdd3 = rdd2.map(word => (word, 1))

    //5, 根据key聚合
    val rdd4 = rdd3.reduceByKey(_ + _)

    //6, foreach
    rdd4.foreach(e => {
      println(e)
    })
  }
}
```



## 2, 取部分结果

### 2.1, first()

```java
package im.ivanl001.bigData.Spark.A03_Action.A02_action_result_part

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 16:43
  * #description :  
  *
  **/
object A0201_action_part_first {

  def main(args: Array[String]): Unit = {

    //创建Spark配置对象
    val conf = new SparkConf()

    //集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]") //数字是本地模式下开启几个线程模拟多线程

    //通过conf创建sc
    val sc = new SparkContext(conf)

    //加载文本文件
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang01.txt", 4) //数字代表分区


    //动作first是取出第一个元素，rdd1中现在是每行文字，所以first也即是取出第一行文字
    println("-------------------------first-------------------------")
    val firstLine = rdd1.first()
    println("获取第一个元素：" + firstLine)
  }
}
```



### 2.2, take(n)

```java
package im.ivanl001.bigData.Spark.A03_Action.A02_action_result_part

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 16:43
  * #description : 
  *
  **/
object A0202_action_part_take {

  def main(args: Array[String]): Unit = {

    //创建Spark配置对象
    val conf = new SparkConf()

    //集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程

    //通过conf创建sc
    val sc = new SparkContext(conf)

    //加载文本文件
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang01.txt", 4)//数字代表分区


    //动作first是取出第一个元素，rdd1中现在是每行文字，所以first也即是取出第一行文字
    println("-------------------------first-------------------------")
    val firstLine = rdd1.first()
    println(firstLine)

    //其实first就是调用的take，我们可以直接调用take动作更好
    println("-------------------------take(n)-------------------------")
    val firstTwoLines = rdd1.take(2)
    firstTwoLines.foreach(println(_))
  }
}
```



### 2.3, takeOrdered(n, [ordering])

```java

```



### 2.4, takeSample(withReplacement,num, [seed])

```java

```



## 3, 取结果聚合

### 3.1, reduce(func)

```java
package im.ivanl001.bigData.Spark.A03_Action.A03_action_result_reduce

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 16:45
  * #description :  
  *
  **/
object A0301_action_reduce_reduce {
  def main(args: Array[String]): Unit = {

    //1, 创建Spark配置对象, 并通过配置对象获取上下文
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程
    //通过conf创建sc
    val sc = new SparkContext(conf)

    //2, 加载文本文件，数字4代表最小分区数，最终分区等于或者大于4
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)
    println(rdd1.count())

    //3, flatMap操作
    val rdd2 = rdd1.flatMap(line => {
      line.split(" ")
    })

    //4, 对比可以看到，rdd1的个数是1，rdd2的个数是6，也就是说flatmap可以从一个输入映射到多个输出
    println("个数是：" + rdd2.count())

    //5, 聚合操作，如果是word，是直接把word拼接起来，如果是int，是加和
    //reduce输入的是什么类型的值，输出的就是什么类型的值，比如输入的是字符串，输出也是字符串，输入的是int，输出的也是int
    val rdd3 = rdd2.reduce(_+_)
    println(new String(rdd3.getBytes))
  }
}

//上面代码其实是对一个字符串数据进行聚合，那么就直接拼接字符串了，所以打印结果是：
1
个数是：6
ivanl001isthekingofworld!
```

### 3.2, count()

```java
package im.ivanl001.bigData.Spark.A03_Action.A03_action_result_reduce

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 16:46
  * #description : 
  *
  **/
object A0302_action_reduce_count {

  def main(args: Array[String]): Unit = {

    //1, 创建Spark配置对象, 并通过配置对象获取上下文
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程
    //通过conf创建sc
    val sc = new SparkContext(conf)

    //2, 加载文本文件，数字4代表最小分区数，最终分区等于或者大于4
    //val rdd1 = sc.textFile(args(0))
    //文本内容：ivanl001 is the king of world!
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)
    println(rdd1.count())

    //3, flatMap操作
    val rdd2 = rdd1.flatMap(line => {
      line.split(" ")
    })

    //4, 对比可以看到，rdd1的个数是1，rdd2的个数是6，也就是说flatmap可以从一个输入映射到多个输出
    println("个数是：" + rdd2.count())
  }
}

// 打印结果是，因为一共6个单词咯
1
个数是：6
```

### 3.3, countByKey()

```java
package im.ivanl001.bigData.Spark.A03_Action.A03_action_result_reduce

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 16:46
  * #description : 
  *
  **/
object A0303_action_reduce_countByKey {

  def main(args: Array[String]): Unit = {

    //1, 创建Spark配置对象, 并通过配置对象获取上下文
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程
    //通过conf创建sc
    val sc = new SparkContext(conf)

    //2, 加载文本文件，数字4代表最小分区数，最终分区等于或者大于4
    //val rdd1 = sc.textFile(args(0))
    //文本内容：ivanl001 is the king of world!
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang.txt", 4)
    println(rdd1.count())

    //3, flatMap操作
    val rdd2 = rdd1.flatMap(line => {
      line.split(" ")
    }).map(word => (word, 1))

    //4,countByKey
    rdd2.countByKey().foreach(println(_))
  }
}
```



## 4, 保存结果

### 4.1, saveAsTextFile(path)

```java
package im.ivanl001.bigData.Spark.A03_Action.A04_action_result_save

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 16:46
  * #description : 
  *
  **/
object A0401_action_save_saveAsTextFile {

  def main(args: Array[String]): Unit = {

    //创建Spark配置对象
    val conf = new SparkConf()

    //集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程

    //通过conf创建sc
    val sc = new SparkContext(conf)

    //加载文本文件
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang01.txt", 4)//数字代表分区


    //保存成文件，和hadoop的一样
    println("-------------------------saveAsTextFile-------------------------")
    rdd1.flatMap(_.split(" ")).saveAsTextFile("/Users/ivanl001/Desktop/00-bigData/00-data/output/textfile/")

  }
}
```



### 4.2, saveAsSequenceFile(path) (Java and Scala)

```java
package im.ivanl001.bigData.Spark.A03_Action.A04_action_result_save

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 16:47
  * #description : 
  *
  **/
object A0402_action_save_saveAsSequenceFile {

  def main(args: Array[String]): Unit = {

    //创建Spark配置对象
    val conf = new SparkConf()

    //集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程

    //通过conf创建sc
    val sc = new SparkContext(conf)

    //加载文本文件
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang01.txt", 4)//数字代表分区


    //保存成序列文件,只有有key-value对的rdd才能保存成序列文件哈
    println("-------------------------saveAsSequenceFile-------------------------")
    //必须是key-value才能保存成序列文件
    rdd1.flatMap(_.split(" ")).map((_,1)).saveAsSequenceFile("/Users/ivanl001/Desktop/00-bigData/00-data/output/seqfile/")

    println("-------------------------加载SequenceFile,暂时不会加载-------------------------")
    /*val rdd2 = sc.sequenceFile("/Users/ivanl001/Desktop/00-bigData/00-data/output/seqfile/")
    rdd2.foreach(println(_))*/
  }
}
```



### 4.3, saveAsObjectFile(path) (Java and Scala)

```java
package im.ivanl001.bigData.Spark.A03_Action.A04_action_result_save

import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-17 16:47
  * #description : 
  *
  **/
object A0403_action_save_saveAsObjectFile {

  def main(args: Array[String]): Unit = {

    //创建Spark配置对象
    val conf = new SparkConf()

    //集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程

    //通过conf创建sc
    val sc = new SparkContext(conf)

    //加载文本文件
    //val rdd1 = sc.textFile(args(0))
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/00-bigData/00-data/input/zhang01.txt", 4)//数字代表分区

    println("-------------------------saveAssaveAsObjectFile-------------------------")
    rdd1.flatMap(_.split(" ")).saveAsObjectFile("/Users/ivanl001/Desktop/00-bigData/00-data/output/objfile/")

    //加载之后就是存储对那个rdd
    println("-------------------------加载ObjectFile-------------------------")
    val objRdd = sc.objectFile("/Users/ivanl001/Desktop/00-bigData/00-data/output/objfile/")
    objRdd.countByValue().foreach(println(_))
  }
}
```







