*storm和hbase的集成最好自己写一个bolt，在bolt中建立hbase连接，通过表进行存入，具体如下*

* 01，配置pom.xml文件 

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

* 02, 实现IRichBolt，写一个hbaseBolt
  ```java
  package im.ivanl001.bigData.Storm.A07_Storm_Hbase;

  import org.apache.hadoop.conf.Configuration;
  import org.apache.hadoop.hbase.HBaseConfiguration;
  import org.apache.hadoop.hbase.TableName;
  import org.apache.hadoop.hbase.client.Connection;
  import org.apache.hadoop.hbase.client.ConnectionFactory;
  import org.apache.hadoop.hbase.client.Table;
  import org.apache.storm.task.OutputCollector;
  import org.apache.storm.task.TopologyContext;
  import org.apache.storm.topology.IRichBolt;
  import org.apache.storm.topology.OutputFieldsDeclarer;
  import org.apache.storm.tuple.Tuple;
  
  import java.io.IOException;
  import java.util.Map;
  
  /**
   * #author      : ivanl001
   * #creator     : 2018-11-24 10:49
   * #description : 发送数据到hbase的bolt节点
   **/
  public class WordCountHbaseBolt implements IRichBolt {
  
      private Connection connection = null;
      private Table table = null;
  
      public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
  
          Configuration configuration = HBaseConfiguration.create();
          try {
              connection = ConnectionFactory.createConnection(configuration);
              table = connection.getTable(TableName.valueOf("storm:t1"));
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  
      public void execute(Tuple tuple) {
  
          String word = tuple.getStringByField("word");
          Integer count = tuple.getIntegerByField("count");
  
          try {
              table.incrementColumnValue(word.getBytes(), "f1".getBytes(), word.getBytes(), count.byteValue());
              System.out.println("+++++++++++" + word + "++++++++++++" + count);
              //因为storm是流计算，不太可能会停，所以这里table可以不必关闭
          } catch (IOException e) {
              e.printStackTrace();
          }
  
      }
  
      public void cleanup() {
  
      }
  
      public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
  
      }
  
      public Map<String, Object> getComponentConfiguration() {
          return null;
      }
  }
  ```
  
* 03, 在app中使用刚才的hbasebolt, 我这里还是使用了二次聚合的方式减少数据倾斜，然后其他几个用到的类这里跟和hbase集成没啥关系，所以就不贴出来了
  ```java
  package im.ivanl001.bigData.Storm.A07_Storm_Hbase;

  import org.apache.storm.Config;
  import org.apache.storm.LocalCluster;
  import org.apache.storm.StormSubmitter;
  import org.apache.storm.generated.AlreadyAliveException;
  import org.apache.storm.generated.AuthorizationException;
  import org.apache.storm.generated.InvalidTopologyException;
  import org.apache.storm.topology.TopologyBuilder;
  import org.apache.storm.tuple.Fields;
  
  /**
   * #author      : ivanl001
   * #creator     : 2018-11-17 09:09
   * #description : 这里集成storm和hbase
   **/
  public class WordCountApp {
  
  
      public static void main(String[] args) throws InterruptedException, InvalidTopologyException, AuthorizationException, AlreadyAliveException {
  
          Config config = new Config();
          config.setDebug(false);
          //这个是设置开启几个worker
          config.setNumWorkers(2);
  
          TopologyBuilder builder = new TopologyBuilder();
  
          //parallelism_hint是设置并发的数量，比如下面spout设置成2,那么会开两个线程，跑3个任务，这个时候一个线程就必须同时跑两个任务，这其实不太好，所以并发数需要尽量大于任务数
          builder.setSpout("wordCountSpout", new WordCountSpout(), 2).setNumTasks(2);
  
          builder.setBolt("wordCountSplitBolt", new WordCountSplitBolt(), 3).shuffleGrouping("wordCountSpout").setNumTasks(3);
  
          //下面的这个hbasebolt其实是逐一递增的，这里的效率其实是不高的，可以先聚合一次，这样子再存到hbase中效率会好很多
          builder.setBolt("wordCountHbaseBolt", new WordCountHbaseBolt(), 1).shuffleGrouping("wordCountSplitBolt").setNumTasks(1);
  
          //这里就不再用默认的类了，而是用hbasebolt类
          //builder.setBolt("wordCountSumBolt", new WordCountSumBolt(), 4).fieldsGrouping("wordCountSplitBolt", new Fields("word")).setNumTasks(4);

          //这里用本地模式来测试分区模式，消息还是发送到master上的nc好了，懒得改了，一样看
          LocalCluster localCluster = new LocalCluster();
          localCluster.submitTopology("localWordCountCluster", config, builder.createTopology());
          /*Thread.sleep(10000);
          localCluster.shutdown();*/
  
          //集群提交
          //StormSubmitter.submitTopology("wordCountCluster", config, builder.createTopology());
      }
  }
  ```