### 1, 首先配置flume

```properties
# 0, 首先指定三种组件：源,通道,槽
agent.sources = r1
agent.channels = c1
agent.sinks = k1


# 1, 定义source相关信息，绑定流入的数据， 并制定输出的方向
# totalEvents可以不必设定
agent.sources.r1.type = spooldir
agent.sources.r1.spoolDir = /data/flume/estore_logs
agent.sources.r1.channels = c1


# 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入
的
agent.channels.c1.type = memory


# 3, 定义sink相关的信息,这里是kafka
agent.sinks.k1.channel = c1
agent.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
agent.sinks.k1.kafka.bootstrap.servers = slave01:9092 slave02:9092 slave03:9092
agent.sinks.k1.kafka.topic = calllogs
agent.sinks.k1.kafka.flumeBatchSize = 20
agent.sinks.k1.kafka.producer.acks = 1
```

### 2， 启动kafka，创建主题

* 01，kafka依赖zk，所以启动kafka之前需要先启动zk
*需要在所选服务器上分别执行如下命令，同时可以通过第二个命令查看zk状态*

  > zkServer.sh start 
  
  > zkServer.sh status
  
* 02, 启动kafka集群
*需要在所选服务器上分别执行如下命令*
  > kafka-server-start.sh /usr/local/kafka/config/server.properties &

* 03, 创建所需要的kafka主题，方便flume把收集到的数据发送过来
*4个分区，三个副本因子，也即是分成4个区，每个区保存3个副本*
  > kafka-topics.sh --create --zookeeper slave01:2181 --replication-factor 3 --partitions 4 --topic estore_logs

* 04, 查看主题
  > kafka-topics.sh --list --zookeeper slave01:2181

* 05, 为了方便测试，我们这里先启动控制台消费者，消费刚才创建的estore_logs主题
  > kafka-console-consumer.sh --bootstrap-server slave01:9092 --topic estore_logs --from-beginning --zookeeper slave02:2181

* 06, 这里顺便给出开启生产者的方式以便调试的时候使用
  > kafka-console-producer.sh --broker-list slave01:9092 --topic estore_logs

* 07，如果有需要，可以设置kafka的存活时间
*server.properties文件*
  ```java
  # The minimum age of a log file to be eligible for deletion
  # 这里是存活时间，默认是一周7*24
  log.retention.hours=168
  ```

### 3, 启动kafka，创建主题后，可以开启flume进行搜集了
> flume-ng agent -n agent -c conf -f /root/flume/conf/myConf/logs-spooldir-kafka.properties &