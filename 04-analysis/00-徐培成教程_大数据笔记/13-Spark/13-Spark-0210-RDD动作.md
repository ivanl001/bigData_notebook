## 2, RDD Action


### 01,collect()

*其实有点不太懂collect的作用了，因为实验了一下，就算没有collect，foreach依然能够取出rdd的值，那要collect做什么嘞-----哈哈哈，明白了， 因为虽然没有用collect，但是用了foreach，foreach也是一个动作！！！*

*Returns all the elements of the dataset as an array at the driver program. This is usually useful after a filter or other operation that returns a sufficiently small subset of the data.*


```scala
println("_________________________________________________")
//val r = rdd4.collect()
//r.foreach(println)
//r.foreach(println(_))

rdd4.foreach(e => {
println("no collect++++++++++++")
println(e)
})
```


### 02, reduce(func)

*注意：reduceByKey虽然是变换，但是reduce是动作哈*

```scala
val rdd4 = rdd3.reduceByKey(_ + _)
//在这里可以再对结果的rdd04再map一次获取所有单词的个数，如下
val rdd5 = rdd4.map(t => t._2)//这里把每个单词的第二个个数取出来，后面再聚合一次就可以得到所有的个数了
val count = rdd5.reduce(_+_)

println("个数是："+count)
```


### 03, count()

*比较简单，就会得出总数*

```scala
//压扁
val rdd2 = rdd1.flatMap(line => {
  println("ivanl001" + line)
  line.split(" ")
})

println("个数是：" + rdd2.count())
```

### 04, 其他的
```scala
package im.ivanl001.bigData.Spark.A03_Action
import org.apache.spark.{SparkConf, SparkContext}

object A0301_Action_first {

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
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/bigData/input/zhang.txt", 4)//数字代表分区


    //动作first是取出第一个元素，rdd1中现在是每行文字，所以first也即是取出第一行文字
    println("-------------------------first-------------------------")
    val firstLine = rdd1.first()
    println(firstLine)

    //其实first就是调用的take，我们可以直接调用take动作更好
    println("-------------------------take-------------------------")
    val firstTwoLines = rdd1.take(2)
    firstTwoLines.foreach(println(_))

    //保存成文件，和hadoop的一样
    println("-------------------------saveAsTextFile-------------------------")
    rdd1.flatMap(_.split(" ")).saveAsTextFile("/Users/ivanl001/Desktop/bigData/input/out/")

    //保存成序列文件,只有有key-value对的rdd才能保存成序列文件哈
    println("-------------------------saveAsSequenceFile-------------------------")
    rdd1.flatMap(_.split(" ")).map((_,1)).saveAsSequenceFile("/Users/ivanl001/Desktop/bigData/input/out01/")

    //countByKey()，也就是计算key出现的次数，注意：这个跟reduceByKey效果上的差别是：如果是 （key,1)这样的，结果一直。但是如果是(key,2)这样的值不是1的，countByKey()可不是总的次数，而是key值出现的次数哈
    println("-------------------------countByKey-------------------------")
    val rdd2 = rdd1.flatMap(_.split(" ")).map((_,3))
    rdd2.countByKey().foreach(println(_))
  }
}
```