## 1，Kafka作为flume的sink，这个是一般的做法，flume收集数据，Kafka做缓冲

* 01,更改配置文件如下，并重命名为flume-netcat-kafka-conf.properties
  ```shell
  # 0, 首先指定三种组件：源,通道,槽
  agent.sources = r1
  agent.channels = c1
  agent.sinks = k1
  
  
  # 1, 定义source相关信息，绑定流入的数据， 并制定输出的方向
  agent.sources.r1.type = netcat
  agent.sources.r1.bind = centos01
  agent.sources.r1.port = 7777
  agent.sources.r1.channels = c1
  
  
  # 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
  agent.channels.c1.type = memory
  
  
  # 3, 定义sink相关的信息
  agent.sinks.k1.channel = c1
  agent.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
  agent.sinks.k1.topic = flume
  agent.sinks.k1.brokerList = centos01:9092
  agent.sinks.k1.requiredAcks = 1
  agent.sinks.k1.batchSize = 20
  agent.sinks.k1.channel = c1
  ```
```
  
* 02,kafka应该是已经启动了的，如果没有启动，请启动kafka

* 03, 使用我们刚才配置的文件启动flume
  
  > flume-ng agent -n agent -c conf -f /usr/local/flume/conf/myConf/flume-netcat-kafka-conf.properties -Dflume.root.logger=INFO,console

-----
下面两个不再练习，直接拷贝一下举例的配置文件

## 2, Kafka作为flume的source，这个很少见

  ```shell
  a1.sources = r1
  a1.sinks = k1
  a1.channels = c1
  
  a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
  a1.sources.r1.batchSize = 5000
  a1.sources.r1.batchDurationMillis = 2000
  a1.sources.r1.kafka.bootstrap.servers = s202:9092
  a1.sources.r1.kafka.topics = test3
  a1.sources.r1.kafka.consumer.group.id = g4
  
  a1.sinks.k1.type = logger
  
  a1.channels.c1.type=memory
  
  a1.sources.r1.channels = c1
  a1.sinks.k1.channel = c1
```

## 3，Kafka作为flume的channel
  ```shell
  a1.sources = r1
  a1.sinks = k1
  a1.channels = c1
  
  a1.sources.r1.type = avro
  a1.sources.r1.bind = localhost
  a1.sources.r1.port = 8888
  
  a1.sinks.k1.type = logger
  
  a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel
  a1.channels.c1.kafka.bootstrap.servers = s202:9092
  a1.channels.c1.kafka.topic = test3
  a1.channels.c1.kafka.consumer.group.id = g6
  
  a1.sources.r1.channels = c1
  a1.sinks.k1.channel = c1
  ```