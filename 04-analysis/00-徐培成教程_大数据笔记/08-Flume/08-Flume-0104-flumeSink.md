## 1, HDFS Sink

*我们这里用nc做源来测试*

* 01,配置文件，配置文件改名为flume-netcat-hdfs-conf.properties

  * 有一点需要注意一下：配置一下hdfs.useLocalTimeStamp为true，要不然会报错什么时间戳是空什么的错误哈
  
  ```shell
  # 0, 首先指定三种组件：源,通道,槽
  agent.sources = r1
  agent.channels = c1
  agent.sinks = k1
  
  # 1, 定义source相关信息，绑定流入的数据， 并制定输出的方向
  agent.sources.r1.type = netcat
  agent.sources.r1.bind = localhost
  agent.sources.r1.port = 8888
  agent.sources.r1.channels = c1
  
  # 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
  agent.channels.c1.type = memory
  
  # 3, 定义sink相关的信息
  agent.sinks.k1.channel = c1 
  agent.sinks.k1.type = hdfs
  agent.sinks.k1.hdfs.path = /flume/events/%y-%m-%d/%H%M/%S
  agent.sinks.k1.hdfs.filePrefix = events-
  # 这里是每隔十分钟创建一个文件夹
  agent.sinks.k1.hdfs.round = true
  agent.sinks.k1.hdfs.roundValue = 10
  agent.sinks.k1.hdfs.roundUnit = minute
  # 每个5seconds或者100个字节进行滚动一个文件，rollCount：Number of events
  agent.sinks.k1.hdfs.rollInterval = 60
  agent.sinks.k1.hdfs.rollSize = 100
  agent.sinks.k1.hdfs.rollCount = 100 
  agent.sinks.k1.hdfs.useLocalTimeStamp = true
  ```
* 02,启动
  
  > flume-ng agent -n agent -c conf -f flume-netcat-hdfs-conf.properties

## 2, HBase Sink

* 01,配置文件，配置文件改名为flume-netcat-hbase-conf.properties
  ```shell
  # 0, 首先指定三种组件：源,通道,槽
  agent.sources = r1
  agent.channels = c1
  agent.sinks = k1
  
  # 1, 定义source相关信息，绑定流入的数据， 并制定输出的方向
  agent.sources.r1.type = netcat
  agent.sources.r1.bind = localhost
  agent.sources.r1.port = 8888
  agent.sources.r1.channels = c1
  
  # 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
  agent.channels.c1.type = memory
  
  # 3, 定义sink相关的信息
  agent.sinks.k1.channel = c1  
  agent.sinks.k1.type = hbase
  agent.sinks.k1.table = flume:t1
  agent.sinks.k1.columnFamily = f1
  ```
  
* 02,启动
  
> flume-ng agent -n agent -c conf -f flume-netcat-hbase-conf.properties

* 报错：18/11/14 12:42:15 ERROR hbase.HBaseSink: Invalid HBase version:2.1.0， 很明显是版本太高的问题， 我这里就不再深究，基本这么配置启动就可以正常的往hbase写入数据了

## 3, Hive Sink 看文档吧

