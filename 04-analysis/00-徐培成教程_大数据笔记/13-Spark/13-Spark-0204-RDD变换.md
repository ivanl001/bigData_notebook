##  1, RDD变换

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

---
---
---



### 01，map(func)

*map(func)和mapPartitions(func)的区别*

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
    ivanl001张丹峰 
    开始---：张丹峰
    结束---:张丹峰
    ivanl001哈哈哈
    开始---：哈哈哈
    结束---:哈哈哈
    ivanl001嘿嘿嘿
    开始---：嘿嘿嘿
    结束---:嘿嘿嘿
    ivanl001厉害啊
    开始---：厉害啊
    结束---:厉害啊
    ivanl001h
    开始---：h
    结束---:h
    ```

### 02，mapPartitions

*然后我们可以先用mapPartitions进行分区区分操作，然后再用map映射处理后的数据就可以了*

如果需要进行第三方数据链接，可以使用mapPartitions，不然使用map会频繁的创建关闭连接，浪费资源的

  ```scala
  val rdd2_1 = rdd2.mapPartitions(it => {

  println("partition start----")
  val buf = ArrayBuffer[String]()

  for (str <- it) {
    println("在mapPartitions中：" + str+ "_posfix")
    buf += str+ "_posfix"
  }

  println("partition end------")

  buf.iterator
  })
  
  //映射w => (w,1)
  val rdd3 = rdd2_1.map(word =>{
    println("开始---：" + word)
    val res = (word, 1)
    println("结束---:" + word)
    res
  })
  
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

### 03，mapPartitionsWithIndex(func)

*mapPartitionsWithIndex和mapPartitions类似,只不过会传递分区索引进来，如下*

  ```scala
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
  ```

### 04,sample(withReplacement, fraction, seed)

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
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/bigData/input/zhang.txt", 4)//数字代表分区

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

### 05,union(otherDataset)

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


### 06, intersection(otherDataset)
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



### 07,distinct([numTasks])) 
*这个可以取出某个rdd中重复的内容*

```scala
package im.ivanl001.bigData.Spark.A02_Transformations

import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0206_WordCountApp_distinct {

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/bigData/input/log.txt",4)
    //所有error
    //val errorRDD = rdd1.filter(_.toLowerCase.contains("error"))

    //所有warn行
    val warnRDD = rdd1.filter(_.toLowerCase.contains("warn"))
    //这里是可以打印出相同的那些行的
    warnRDD.foreach(println)

    println("--------------------------------------")
    
    //这里可以通过distinct去掉某个rdd中重复的部分
    //下面经过去重之后就不会打印出相同的行了，相同的行只能打印出一行哈
    val distinctedWarnRDD = warnRDD.distinct()
    distinctedWarnRDD.foreach(println)
  }
}
```

### 08, groupByKey([numTasks])
*根据key分组,比较好理解，看代码*

> (K, V) => (K, Iterable<V>)
```scala
package im.ivanl001.bigData.Spark.A02_Transformations
import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0207_WordCountApp_groupByKey {

  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/bigData/input/students.txt",1)

    //这里把原先的一行数据映射成(省份,行数据)格式
    val rdd2 = rdd1.map(line => {
      val key = line.split(" ")(2)
      (key, line)
    })

    //因为groupByKey把结果转换成(K, V) => (K, Iterable)这种，所以按照如下方式进行打印
    val rdd3 = rdd2.groupByKey()
    rdd3.foreach(e => {
      val key = e._1
      val iterable = e._2
      for (res <- iterable) {
        println(key + ":" + res)
      }
      println("=====================")
    })
  }
}
//结果如下:这个是根据key分组后的结果，注意看代码中分组后的返回值rdd3中的value是iterable
/*
beijing:4 ivanl004 beijing
beijing:8 ivanl008 beijing
=====================
shanghai:3 ivanl003 shanghai
shanghai:7 ivanl007 shanghai
=====================
henan:1 ivanl001 henan
henan:2 ivanl002 henan
henan:6 ivanl006 henan
=====================
shangdong:5 ivanl005 shangdong
=====================
*/
```

### 09, aggregateByKey(zeroValue)(seqOp, combOp, [numTasks])

​    具体看面试复习里面的spark部分内容

### 10, sortByKey([ascending], [numTasks])
*很明显，就是排序, 代码如下:*
```scala
package im.ivanl001.bigData.Spark.A02_Transformations

import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0208_WordCountApp_sortByKey {

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/bigData/input/students.txt",1)

    //这里把原先的一行数据映射成(省份,行数据)格式
    val rdd2 = rdd1.map(line => {
      val key = line.split(" ")(2)
      (key, line)
    })

    //因为groupByKey把结果转换成(K, V) => (K, Iterable)这种，所以按照如下方式进行打印
    val rdd3 = rdd2.sortByKey()
    rdd3.foreach(e => {
      val key = e._1
      val value = e._2

      println(key + "========" + value)
    })
  }
}
//结果如下:这个是根据key排序后的结果
/*
beijing========4 ivanl004 beijing
beijing========8 ivanl008 beijing
henan========1 ivanl001 henan
henan========2 ivanl002 henan
henan========6 ivanl006 henan
shangdong========5 ivanl005 shangdong
shanghai========3 ivanl003 shanghai
shanghai========7 ivanl007 shanghai
*/
```


### 11, join(otherDataset, [numTasks])
*上面05中已经讲过union，union就是把两个rdd的内容合并到一个rdd中去，是纵向的，join不一样，join是横向的*
*方式大概是: (K, V) and (K, W)  =>  (K, (V, W)),看起来和groupByKey有点像对吧，但是前者是针对多张表进行的join，而group是在单张表中进行分组的哈*

```scala
package im.ivanl001.bigData.Spark.A02_Transformations
import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0209_WordCountApp_join {

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)
    
    val customerRdd = sc.textFile("/Users/ivanl001/Desktop/bigData/input/customer.txt",1)
    val ordersRdd = sc.textFile("/Users/ivanl001/Desktop/bigData/input/orders.txt",1)
    
    val customerRdd2 = customerRdd.map(line => (line.split(" ")(0),line))
    val orderRdd2 = ordersRdd.map(line => (line.split(" ")(0),line))

    val rdd = customerRdd2.join(orderRdd2)
    rdd.foreach(e => {
      val key = e._1
      val value = e._2

      println(key + ":::::" + value)
    })
  }
}
//结果如下
/*
2:::::(2 ivanl002,2 ae)
3:::::(3 ivanl003,3 西红柿)
3:::::(3 ivanl003,3 苹果)
1:::::(1 ivanl001,1 eos5dMark4)
1:::::(1 ivanl001,1 iphone)
1:::::(1 ivanl001,1 finalCut)
*/
```

### 12, cogroup(otherDataset, [numTasks])
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

### 13，cartesian(otherDataset)
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
### 13，pipe(command, [envVars])
*这个可以执行shell脚本命令，把结果包装成rdd返回*
```scala
package im.ivanl001.bigData.Spark.A02_Transformations
import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0212_WordCountApp_pipe {

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[4]") ;
    val sc = new SparkContext(conf)

    val rdd01 = sc.parallelize("/Users/ivanl001/Desktop/")
    //这个在linux下可以，在mac或者wins下好像不好使
    //val rdd02 = rdd01.pipe("ls")
    val rdd02 = rdd01.pipe("ls /Users/ivanl001/Desktop/")
    rdd02.foreach(println(_))
  }
}
```

### 14, coalesce(numPartitions)和repartition(numPartitions)
*前者是减少分区，后者是重新分区，可以减少分区，也能增加分区*

```scala
package im.ivanl001.bigData.Spark.A02_Transformations
import org.apache.spark.{SparkConf, SparkContext}

//wordcount主程序
object A0213_WordCountApp_coalesce {

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
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/bigData/input/zhang.txt", 5)//数字代表分区
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
```

### 15, repartitionAndSortWithinPartitions(partitioner)

*重新分区并在分区内进行排序*

