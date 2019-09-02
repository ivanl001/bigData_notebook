## 1, Storm和kafka集成
*注意：storm和kafka的集成不需要服务器上特别的配置，也即是代码层面的，从kafka中发送到storm里面即可*

集成的时候总是在报错， 什么不能同步supervisor什么的，或者循环死掉了， log4j啥的问题，就是log4j需要单独指定， 然后版本也需要搭配，看pom.xml


* 01, 首先配置pom.xml的依赖，注意：不能使用默认的log4j，会报错，需要单独指定，如下
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>im.ivanl001</groupId>
      <artifactId>10-Storm</artifactId>
      <version>1.0-SNAPSHOT</version>
  
      <packaging>jar</packaging>
  
      <!--这他妈要求也太高了， 版本不对也不行，这怎么玩，要是升级了我怎么知道是哪个版本的？？？？？-->
      <dependencies>
          <dependency>
              <groupId>org.apache.storm</groupId>
              <artifactId>storm-core</artifactId>
              <version>1.0.3</version>
          </dependency>
          <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.11</version>
          </dependency>
          <dependency>
              <groupId>org.apache.storm</groupId>
              <artifactId>storm-kafka</artifactId>
              <version>1.0.2</version>
          </dependency>
          <dependency>
              <groupId>log4j</groupId>
              <artifactId>log4j</artifactId>
              <version>1.2.17</version>
          </dependency>
          <dependency>
              <groupId>org.apache.kafka</groupId>
              <artifactId>kafka_2.10</artifactId>
              <version>0.8.1.1</version>
              <exclusions>
                  <exclusion>
                      <groupId>org.apache.zookeeper</groupId>
                      <artifactId>zookeeper</artifactId>
                  </exclusion>
                  <exclusion>
                      <groupId>log4j</groupId>
                      <artifactId>log4j</artifactId>
                  </exclusion>
              </exclusions>
          </dependency>
  
          <dependency>
              <groupId>org.apache.storm</groupId>
              <artifactId>storm-hbase</artifactId>
              <version>1.0.3</version>
          </dependency>
  
          <dependency>
              <groupId>org.apache.hadoop</groupId>
              <artifactId>hadoop-common</artifactId>
              <version>2.7.3</version>
          </dependency>
  
      </dependencies>
  
      <build>
          <finalName>Storm_test</finalName>
          <!--这个会输出包中的内容到这个文件夹-->
          <!--<outputDirectory>/Users/ivanl001/Desktop/everything/</outputDirectory>-->
          <!--这个会把打好的包放在下面这个文件夹-->
          <directory>/Users/ivanl001/Desktop/everything/jar/</directory>
      </build>
  
  
  </project>
  ```
* 02, 需要配置kafka_spout, 这个是自带的，不需要自己重新写，如下：

  ```java
  package im.ivanl001.bigData.Storm.A06_Storm_Kafka;
  
  import org.apache.storm.Config;
  import org.apache.storm.LocalCluster;
  import org.apache.storm.StormSubmitter;
  import org.apache.storm.generated.AlreadyAliveException;
  import org.apache.storm.generated.AuthorizationException;
  import org.apache.storm.generated.InvalidTopologyException;
  import org.apache.storm.kafka.*;
  import org.apache.storm.spout.SchemeAsMultiScheme;
  import org.apache.storm.topology.TopologyBuilder;
  
  import java.util.UUID;
  
  /**
   * #author      : ivanl001
   * #creator     : 2018-11-17 09:09
   * #description : 单词统计app, 和kafka的集成, 集成还有问题，先放着吧
   **/
  public class WordCountApp {
  
  
      public static void main(String[] args) throws InterruptedException, InvalidTopologyException, AuthorizationException, AlreadyAliveException {
  
          Config config = new Config();
          config.setDebug(false);
          //这个是设置开启几个worker
          config.setNumWorkers(1);
  
          TopologyBuilder builder = new TopologyBuilder();
  
          //这个是老api的写法，如果看官方文档可能和这个不太一样，但是使用的版本也是不一样的，需要适当的参考
          String zkConnectString = "slave01:2181";
          BrokerHosts hosts = new ZkHosts(zkConnectString);
          //zkRoot不知道填写什么，先空着
          SpoutConfig spoutConfig = new SpoutConfig(hosts, "storm", "/storm", UUID.randomUUID().toString());
          spoutConfig.scheme = new SchemeAsMultiScheme(new StringScheme());
          KafkaSpout kafkaSpout = new KafkaSpout(spoutConfig);
  
          //这里设置spout
          builder.setSpout("kafka_spout", kafkaSpout, 1).setNumTasks(1);
  
          //这里设置bolt
          builder.setBolt("wordCountSplitBolt", new WordCountSplitBolt(), 1).shuffleGrouping("kafka_spout").setNumTasks(1);
  
          LocalCluster localCluster = new LocalCluster();
          localCluster.submitTopology("localWordCountCluster", config, builder.createTopology());
          /*Thread.sleep(10000);
          localCluster.shutdown();*/
  
          //集群提交
          //StormSubmitter.submitTopology("wordCountCluster", config, builder.createTopology());
      }
  }
  
  ```