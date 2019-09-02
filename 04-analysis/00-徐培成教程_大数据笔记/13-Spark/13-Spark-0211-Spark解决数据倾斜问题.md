*思路分析：*
*其实就是二次聚合，第一次聚合前key添加随机后缀，第二次再去掉后缀进行聚合*
*在spark中，解决数据倾斜的问题，方法其实也很简单：第一次map前，在key值后面添加特定的后缀，比如说有10个分区，那么随机添加后缀"_0"到"_9",这样的话，就算是相同的key在添加后缀后，也能被随机的分配到不同的分区上，第一次聚合后，重新map，把后缀去掉，再重新把结果聚合一下*

```scala
package im.ivanl001.bigData.Spark.A04_DataLean
import org.apache.spark.{SparkConf, SparkContext}
import scala.util.Random

//wordcount主程序
object A0401_WordCountApp_NoDataLean {

  def main(args: Array[String]): Unit = {

    //1, 基本对象的配置和创建
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程
    val sc = new SparkContext(conf)


    //2, 加载文本文件
    val rdd1 = sc.textFile("/Users/ivanl001/Desktop/bigData/input/zhang.txt", 4)//数字代表分区

    //3, 压扁
    val rdd2 = rdd1.flatMap(line => {
      line.split(" ")
    })

    //4, 映射w => (w,1),这里为了减少数据倾斜，添加随机后缀
    val rdd3 = rdd2.map(word =>{
      val postFix = "_" + Random.nextInt(4)
      (word + postFix, 1)
    })

    //计算
    val rdd4 = rdd3.reduceByKey(_ + _)
    rdd4.foreach(println(_))
    rdd4.saveAsTextFile("/Users/ivanl001/Desktop/bigData/input/nodataLeanOut01/")

    rdd4.map(e => {
      (e._1.split("_")(0),e._2)
    }).reduceByKey(_ + _).saveAsTextFile("/Users/ivanl001/Desktop/bigData/input/nodataLeanOut02/")
  }
}
```