## 1，Stream模块的使用
* 01, 添加maven依赖
  ```xml
  <!--spark-stream计算需要的架包-->
  <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-streaming_2.11</artifactId>
      <version>2.1.0</version>
  </dependency>

  <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
      <version>2.1.0</version>
  </dependency>
  ```
* 02, 编写代码
  *scala代码*
  ```scala
  package im.ivanl001.bigData.Spark.A10_SparkStream

  import org.apache.spark.SparkConf
  import org.apache.spark.streaming.{Seconds, StreamingContext}
  
  object A1001_SparkStreamDemo {
  
    def main(args: Array[String]): Unit = {
  
      //01, 创建Spark配置对象
      /*val conf = new SparkConf()
      //集群模式下下面两行不要
      conf.setAppName("WordCountScala")
      //设置master属性
      //conf.setMaster("spark://master:7077")
      conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程*/
  
  
      //01, 下面我们用第二种方式创建配置对象,有一点需要注意：流处理是在分线程执行，所以这里必须设置大于1条线程才行
      //val conf = new SparkConf().setAppName("streamDemo").setMaster("local[2]")
      //分布式下的设置
      val conf = new SparkConf().setAppName("streamDemo").setMaster("spark://master:7077")
  
      //02, 创建流对象上下文,注意设置间隔时间哦
      val streamContext = new StreamingContext(conf, Seconds(10))//分布式下设置成10秒吧
  
      //03,创建nc文本流用于接受数据
      // Create a DStream that will connect to hostname:port, like localhost:9999
      val lines = streamContext.socketTextStream("master", 9999)
  
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
  *java代码*
  ```java
  package im.ivanl001.bigData.Spark.A10_SparkStream;

  import org.apache.spark.SparkConf;
  import org.apache.spark.api.java.function.FlatMapFunction;
  import org.apache.spark.api.java.function.Function2;
  import org.apache.spark.api.java.function.PairFunction;
  import org.apache.spark.streaming.Seconds;
  import org.apache.spark.streaming.api.java.JavaDStream;
  import org.apache.spark.streaming.api.java.JavaPairDStream;
  import org.apache.spark.streaming.api.java.JavaReceiverInputDStream;
  import org.apache.spark.streaming.api.java.JavaStreamingContext;
  import scala.Tuple2;
  
  import java.util.Arrays;
  import java.util.Iterator;
  
  
  /**
   * #author      : ivanl001
   * #creator     : 2018-12-01 14:19
   * #description : java版本
   **/
  public class A1001_SparkStreamDemo_java {
  
      public static void main(String[] args) throws Exception {
  
  
          //01, 创建配置对象
          SparkConf conf = new SparkConf();
          conf.setAppName("java_stream");
          //conf.setMaster("local[2]");
          conf.setMaster("spark://master:7077");
  
          //02, 创建流对象上下文,注意设置间隔时间哦
          JavaStreamingContext streamingContext = new JavaStreamingContext(conf, Seconds.apply(10));
  
          //03, 创建nc文件输入
          JavaReceiverInputDStream inputDStream =  streamingContext.socketTextStream("master", 9999);
  
          //04, 切割数据, 这里的JavaReceiverInputDStream可以使用flatMap方法不是特别懂
          /*JavaDStream<String> wordDS = inputDStream.flatMap(new FlatMapFunction<String, String>() {
               @Override
               public Iterator<String> call(String s) throws Exception {
                   System.out.println("ivanl001--");
                   List<String> flatList = new ArrayList<String>();
                   String[] splitedW = s.split(" ");
                   for (String word : splitedW) {
                       flatList.add(word);
                   }
                   return flatList.iterator();
               }
           });*/
          //上面的和这里的是相同的
          JavaDStream<String> words = inputDStream.flatMap(
              new FlatMapFunction<String, String>() {
                  @Override public Iterator<String> call(String x) {
                      return Arrays.asList(x.split(" ")).iterator();
                  }
              });
  
          //05,映射后进行聚合并打印
          JavaPairDStream<String, Integer> pairDStream = words.mapToPair(new PairFunction<String, String, Integer>() {
              @Override
              public Tuple2<String, Integer> call(String s) throws Exception {
                  return new Tuple2<>(s, 1);
              }
          });
  
          JavaPairDStream<String, Integer> resultPair = pairDStream.reduceByKey(new Function2<Integer, Integer, Integer>() {
              @Override
              public Integer call(Integer v1, Integer v2) throws Exception {
                  return v1+v2;
              }
          });
  
          //06,打印
          resultPair.print();
          streamingContext.start();              // Start the computation
          streamingContext.awaitTermination();   // Wait for the computation to terminate
      }
  }
  ```
* 03, 提交到服务器上运行
  > spark-submit --master spark://master:7077 --name stream01  --class im.ivanl001.bigData.Spark.A10_SparkStream.A1001_SparkStreamDemo Spark_Test.jar

  > spark-submit --master spark://master:7077 --name stream01  --class im.ivanl001.bigData.Spark.A10_SparkStream.A1001_SparkStreamDemo_java Spark_Test.jar