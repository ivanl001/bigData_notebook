## 1, netcat

* 01,配置文件，配置文件改名为flume-netcat-conf.properties
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
  agent.sinks.k1.type = logger
  ```
  
* 02,启动
  
  > flume-ng agent -n agent -c conf -f /usr/local/flume/conf/myConf/flume-netcat-conf.properties



## 2，Sequence Generator Source，这个案例的具体用途不是很明白

  *A simple sequence generator that continuously generates events with a counter that starts from 0, increments by 1 and stops at totalEvents. Retries when it can’t send events to the channel. Useful mainly for testing.*

* 01, 配置文件，重新命名为flume-seq-conf.properties
  ```shell
  # 0, 首先指定三种组件：源,通道,槽
  agent.sources = r1
  agent.channels = c1
  agent.sinks = k1
  
  
  # 1, 定义source相关信息，绑定流入的数据， 并制定输出的方向
  # totalEvents可以不必设定
  agent.sources.r1.type = seq
  agent.sources.r1.totalEvents = 1000
  agent.sources.r1.channels = c1
  
  
  # 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
  agent.channels.c1.type = memory
  
  
  # 3, 定义sink相关的信息
  agent.sinks.k1.channel = c1
  agent.sinks.k1.type = logger
  ```
  
* 02, 启动, -Dflume.root.logger=INFO,console这个加不加好像都行
  > flume-ng agent -n agent -c conf -f flume-seq-conf.properties -Dflume.root.logger=INFO,console
  
## 3，Spooling Directory Source，也就是检测文件夹

*This source lets you ingest data by placing files to be ingested into a “spooling” directory on disk. This source will watch the specified directory for new files, and will parse events out of new files as they appear.*

* 01, 配置文件，重新命名为flume-spooldir-conf.properties
  ```shell
  # 0, 首先指定三种组件：源,通道,槽
  agent.sources = r1
  agent.channels = c1
  agent.sinks = k1
  
  
  # 1, 定义source相关信息，绑定流入的数据， 并制定输出的方向
  agent.sources.r1.type = spooldir
  agent.sources.r1.spoolDir = /data/flume/testData
  agent.sources.r1.channels = c1
  
  
  # 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
  agent.channels.c1.type = memory
  
  
  # 3, 定义sink相关的信息
  agent.sinks.k1.channel = c1
  agent.sinks.k1.type = logger
  
  
  ```
  
* 02, 启动, -Dflume.root.logger=INFO,console这个加不加好像都行
  
  > flume-ng agent -n agent -c conf -f flume-spooldir-conf.properties -Dflume.root.logger=INFO,console

## 4, Exec Source, 实时收集

* 01, 配置文件，重新命名为flume-exec-conf.properties
  
* ⚠️注意：这里一旦down机是可能丢失数据的
  
  ```shell
  # 0, 首先指定三种组件：源,通道,槽
  agent.sources = r1
  agent.channels = c1
  agent.sinks = k1
  
  
  # 1, 定义source相关信息，绑定流入的数据， 并制定输出的方向
  agent.sources.r1.type = exec
  agent.sources.r1.command = tail -f /data/flume/test/log4j.properties
  agent.sources.r1.channels = c1
  
  
  # 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
  agent.channels.c1.type = memory
  
  
  # 3, 定义sink相关的信息
  agent.sinks.k1.channel = c1
  agent.sinks.k1.type = logger
  ```
  
* 02, 启动, -Dflume.root.logger=INFO,console这个加不加好像都行
  
  > flume-ng agent -n agent -c conf -f flume-exec-conf.properties -Dflume.root.logger=INFO,console

## 5，Stress Source
*这个是压力源，应该主要是用来做压力测试的， 我也不懂，先放着*

* 01, 配置文件，重新命名为flume-stress-conf.properties
  ```shell
  # 0, 首先指定三种组件：源,通道,槽
  agent.sources = r1
  agent.channels = c1
  agent.sinks = k1
  
  
  # 1, 定义source相关信息，绑定流入的数据， 并制定输出的方向
  agent.sources.r1.type = org.apache.flume.source.StressSource
  a1.sources.r1.size = 10240
  a1.sources.r1.maxTotalEvents = 1000000
  a1.sources.r1.channels = memoryChannel-1
  agent.sources.r1.channels = c1
  
  
  # 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
  agent.channels.c1.type = memory
  
  
  # 3, 定义sink相关的信息
  agent.sinks.k1.channel = c1
  agent.sinks.k1.type = logger
  ```
  
* 02, 启动, -Dflume.root.logger=INFO,console这个加不加好像都行
  
  > flume-ng agent -n agent -c conf -f flume-exec-conf.properties -Dflume.root.logger=INFO,console

