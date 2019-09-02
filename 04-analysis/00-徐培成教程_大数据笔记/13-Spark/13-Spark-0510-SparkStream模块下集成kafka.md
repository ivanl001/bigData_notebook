```shell
kafka-console-producer --broker-list centos01:9092 --topic spark01

kafka-console-consumer --bootstrap-server centos01:9092 --topic spark01 --from-beginning
```







* 01，首先添加maven依赖
  
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
  
* 02, 代码如下: 运行之前先去kafka创建主题并开启kafka的生产者

  > kafka-console-producer.sh --broker-list slave01:9092 --topic test

  ```java
  package im.ivanl001.bigData.Spark.A11_SparkStream_kafka;
  
  import org.apache.kafka.clients.consumer.ConsumerRecord;
  import org.apache.kafka.common.serialization.StringDeserializer;
  import org.apache.spark.SparkConf;
  import org.apache.spark.api.java.function.FlatMapFunction;
  import org.apache.spark.api.java.function.Function2;
  import org.apache.spark.api.java.function.PairFunction;
  import org.apache.spark.streaming.Seconds;
  import org.apache.spark.streaming.api.java.JavaDStream;
  import org.apache.spark.streaming.api.java.JavaInputDStream;
  import org.apache.spark.streaming.api.java.JavaPairDStream;
  import org.apache.spark.streaming.api.java.JavaStreamingContext;
  import org.apache.spark.streaming.kafka010.ConsumerStrategies;
  import org.apache.spark.streaming.kafka010.KafkaUtils;
  import org.apache.spark.streaming.kafka010.LocationStrategies;
  import scala.Tuple2;
  
  import java.util.*;
  
  /**
   * #author      : ivanl001
   * #creator     : 2018-12-01 17:52
   * #description : java版的集成kafka
   **/
  public class A1101_Stream_kafka_java {
  
      public static void main(String[] args) throws Exception {
  
  
          //01, 创建配置对象
          SparkConf conf = new SparkConf();
          conf.setAppName("java_stream");
          conf.setMaster("local[2]");
          //conf.setMaster("spark://master:7077");
  
          //02, 创建流对象上下文,注意设置间隔时间哦
          JavaStreamingContext streamingContext = new JavaStreamingContext(conf, Seconds.apply(10));
  
          //03, 配置kafka的参数
          Map<String, Object> kafkaParams = new HashMap<>();
          kafkaParams.put("bootstrap.servers", "slave01:9092,slave02:9092,slave03:9092");
          kafkaParams.put("key.deserializer", StringDeserializer.class);
          kafkaParams.put("value.deserializer", StringDeserializer.class);
          kafkaParams.put("group.id", "use_a_separate_group_id_for_each_stream");
          kafkaParams.put("auto.offset.reset", "latest");
          kafkaParams.put("enable.auto.commit", false);
          //这里可以接受多个主题，不过我们这里就暂时还接受一个就足够了
          //Collection<String> topics = Arrays.asList("topicA", "topicB");
          Collection<String> topics = Arrays.asList("test");
  
  
          JavaInputDStream<ConsumerRecord<String, String>> inputDStream =
                  KafkaUtils.createDirectStream(
                          streamingContext,
                          LocationStrategies.PreferConsistent(),
                          ConsumerStrategies.<String, String>Subscribe(topics, kafkaParams)
                  );
  
  
          //这种方式不是很好处理
          /*JavaPairDStream<String, String> pairDStream = stream.mapToPair(
                  new PairFunction<ConsumerRecord<String, String>, String, String>() {
  
                      @Override
                      public Tuple2<String, String> call(ConsumerRecord<String, String> record) {
  
                          //这里其实只有一个value，是没有key的哈, 这里的value是输入的一行文字
                          System.out.println("the key is :" + record.key() + "-------, the value is : " + record.value());
  
                          return new Tuple2<>(record.key(), record.value());
                      }
                  });*/
  
          //所以我们用第二种方式
          JavaDStream<String> words = inputDStream.flatMap(new FlatMapFunction<ConsumerRecord<String, String>, String>() {
              @Override
              public Iterator<String> call(ConsumerRecord<String, String> stringStringConsumerRecord) throws Exception {
                  String value = stringStringConsumerRecord.value();
                  return Arrays.asList(value.split(" ")).iterator();
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
  
          resultPair.print();
  
          streamingContext.start();
          streamingContext.awaitTermination();
      }
  }
  ```
  

03, 在kafka的生产者中进行输入数据即可在控制台看到处理结果
