DStream

## 1, DStream

```shell
# 离散流
DStream: discretized stream, 分散的小批量流。

flink中有两个概念：DataStream和DataSet
DataStream是处理流数据
DataSet是处理批数据。

而spark中的DStream其实也是小批量，也是批量处理，类似于flink中的DataSet。
```



## 2, spark实时计算代码

```scala
package im.ivanl001.bigData.Spark.A09_SparkStream

import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-18 19:44
  * #description : on-yarn演示
  *
  **/
object A0901_SparkStreamDemo_on_yarn {

  def main(args: Array[String]): Unit = {

    //01, 下面我们用第二种方式创建配置对象,有一点需要注意：流处理是在分线程执行，所以这里必须设置大于1条线程才行
    val conf = new SparkConf().setAppName("streamDemo").setMaster("yarn")
    //分布式下的设置
    //val conf = new SparkConf().setAppName("streamDemo").setMaster("spark://master:7077")

    //02, 创建流对象上下文,注意设置间隔时间哦
    val streamContext = new StreamingContext(conf, Seconds(10))//分布式下设置成10秒吧

    //03,创建nc文本流用于接受数据
    // Create a DStream that will connect to hostname:port, like localhost:9999
    //开启nc: nc -l 9999
    val lines = streamContext.socketTextStream("centos01", 9999)

    //04, 切割数据
    val words = lines.flatMap(_.split(" "))

    //05，映射后进行聚合并打印
    // Count each word in each batch
    val pairs = words.map(word => (word, 1))
    val wordCounts = pairs.reduceByKey(_ + _)

    // Print the first ten elements of each RDD generated in this DStream to the console
    wordCounts.print()

    //06, 正式开始程序等
    streamContext.start()             // Start the computation
    streamContext.awaitTermination()  // Wait for the computation to terminate
  }
}
```

