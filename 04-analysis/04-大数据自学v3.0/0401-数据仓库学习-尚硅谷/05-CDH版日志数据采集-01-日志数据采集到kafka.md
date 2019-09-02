## 1, 数据搜集flume配置

* 下面这个配置是flume-1.6.0版本的哦

```properties
# 0，定义组件
a1.sources=r1
a1.channels=c1 c2 
a1.sinks=k1 k2 

# 1，配置source， configure source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /tmp/logs/app-2019-08-04.log
# Whether to add a header storing the absolute path filename.
# 是否添加header
a1.sources.r1.fileHeader = true
a1.sources.r1.channels = c1 c2

# 2，配置拦截器，interceptor
a1.sources.r1.interceptors = i1 i2
a1.sources.r1.interceptors.i1.type = com.atguigu.flume.interceptor.LogETLInterceptor$Builder
a1.sources.r1.interceptors.i2.type = com.atguigu.flume.interceptor.LogTypeInterceptor$Builder

# 3，通过拦截器代码，配置通道选择器，selector
a1.sources.r1.selector.type = multiplexing
a1.sources.r1.selector.header = topic
a1.sources.r1.selector.mapping.topic_start = c1
a1.sources.r1.selector.mapping.topic_event = c2

# 4，配置通道，configure channel
a1.channels.c1.type = memory
a1.channels.c1.capacity=10000
a1.channels.c1.byteCapacityBufferPercentage=20

a1.channels.c2.type = memory
a1.channels.c2.capacity=10000
a1.channels.c2.byteCapacityBufferPercentage=20

# 5，配置sink，configure sink
# 5.1，第一个sink，进入一个kafka主题：topic_start中
a1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
a1.sinks.k1.kafka.topic = topic_start
a1.sinks.k1.kafka.bootstrap.servers = centos01:9092,centos02:9092,centos03:9092
a1.sinks.k1.kafka.flumeBatchSize = 2000
a1.sinks.k1.kafka.producer.acks = 1
a1.sinks.k1.channel = c1

# 5.2，第二个sink，进入另外一个kafka主题：topic_event中
a1.sinks.k2.type = org.apache.flume.sink.kafka.KafkaSink
a1.sinks.k2.kafka.topic = topic_event
a1.sinks.k2.kafka.bootstrap.servers = centos01:9092,centos02:9092,centos03:9092
a1.sinks.k2.kafka.flumeBatchSize = 2000
a1.sinks.k2.kafka.producer.acks = 1
a1.sinks.k2.channel = c2
```

## 2, kafka主题监听命令

```shell
kafka-console-consumer --bootstrap-server centos01:9092,centos02:9092,centos03:9092 --topic topic_event --group test01
```

## 3, 数据模拟程序启动命令

```shell
# $1是消息之间的时间间隔, $2是条数
java -jar /root/06-dataware/01-log-collector-1.0-SNAPSHOT-jar-with-dependencies.jar  $1 $2>> /root/06-dataware/test.log
# 注意：我们用的是spooldir，只能检测文件夹下的不同的文件名的文件的产生，所以如果需要重复测试，可以拷贝原先的文件更改名字才能被识别
```

## 4, 消费flume配置

> ⚠️注意：如果hdfs中频繁滚动产生很多小文件，可以参考如下文章：
>
> 解决办法：a1.sinks.k1.hdfs.minBlockReplicas = 1
>
> https://www.cnblogs.com/haoyy/p/6097264.html
>
> https://blog.csdn.net/weixin_33910460/article/details/85922140
>
> http://www.it610.com/article/2107322.htm

```properties
## 组件
a1.sources = r1 r2
a1.channels = c1 c2
a1.sinks = k1 k2

## source1
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.zookeeperConnect = centos01:2181,centos02:2181,centos03:2181
a1.sources.r1.topic = topic_start
a1.sources.r1.groupId = flume
a1.sources.r1.batchSize = 5000
a1.sources.r1.batchDurationMillis = 2000

## source2
a1.sources.r2.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r2.zookeeperConnect = centos01:2181,centos02:2181,centos03:2181
a1.sources.r2.topic = topic_event
a1.sources.r2.groupId = flume
a1.sources.r2.batchSize = 5000
a1.sources.r2.batchDurationMillis = 2000


## channel1
a1.channels.c1.type = memory
a1.channels.c1.capacity = 100000
a1.channels.c1.transactionCapacity = 10000

## channel2
a1.channels.c2.type = memory
a1.channels.c2.capacity = 100000
a1.channels.c2.transactionCapacity = 10000

## sink1
a1.sinks.k1.type = hdfs
# a1.sinks.k1.hdfs.writeFormat = Text
a1.sinks.k1.hdfs.useLocalTimeStamp = true
a1.sinks.k1.hdfs.path = /flume/events/log/topic_start/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = logstart-
a1.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.roundValue = 10
a1.sinks.k1.hdfs.roundUnit = second

##sink2
a1.sinks.k2.type = hdfs
# a1.sinks.k2.hdfs.writeFormat = Text
a1.sinks.k2.hdfs.useLocalTimeStamp = true
a1.sinks.k2.hdfs.path = /flume/events/log/topic_event/%Y-%m-%d
a1.sinks.k2.hdfs.filePrefix = logevent-
a1.sinks.k2.hdfs.round = true
a1.sinks.k2.hdfs.roundValue = 10
a1.sinks.k2.hdfs.roundUnit = second


## 不要产生大量小文件
a1.sinks.k1.hdfs.minBlockReplicas = 1
a1.sinks.k1.hdfs.callTimeout = 100000
a1.sinks.k1.hdfs.rollInterval = 60
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0

a1.sinks.k2.hdfs.minBlockReplicas = 1
a1.sinks.k2.hdfs.callTimeout = 100000
a1.sinks.k2.hdfs.rollInterval = 60
a1.sinks.k2.hdfs.rollSize = 134217728
a1.sinks.k2.hdfs.rollCount = 0

## 控制输出文件是原生文件。
# a1.sinks.k1.hdfs.fileType = DataStream 
# a1.sinks.k2.hdfs.fileType = DataStream 
a1.sinks.k1.hdfs.fileType = CompressedStream 
a1.sinks.k2.hdfs.fileType = CompressedStream 

a1.sinks.k1.hdfs.codeC = lzop
a1.sinks.k2.hdfs.codeC = lzop

## 拼装
a1.sources.r1.channels = c1
a1.sinks.k1.channel= c1

a1.sources.r2.channels = c2
a1.sinks.k2.channel= c2
```

