

*主要的意思是：从一个flume的agent:a1出来后，进入到另外一个agent:a2里面去，这种就需要设置a1的sink是arvo，a2的source是arvo*
*a1:端口设置为9901，a2的端口设置成9902，需要先开a2，具体配置操作如下：*

#### 1，修改配置文件， 重命名为flume-hop-conf.properties
  ```shell
  # -----------------首先先定义第一个agent：a1-----------------
  
  # 0, 首先指定三种组件：源,通道,槽
  a1.sources = r1
  a1.channels = c1
  a1.sinks = k1
  
  
  # 1, 定义source相关信息，绑定流入的数据， 并制定输出的方向
  a1.sources.r1.type = netcat
  a1.sources.r1.bind = localhost
  a1.sources.r1.port = 9901
  a1.sources.r1.channels = c1
  
  
  # 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
  a1.channels.c1.type = memory
  
  
  # 3, 定义sink相关的信息
  a1.sinks.k1.channel = c1
  
  a1.sinks.k1.type = avro
  a1.sinks.k1.hostname = localhost
  a1.sinks.k1.port = 9902
  
  
  # -----------------再定义第一个agent：a2-----------------
  
  # 0, 首先指定三种组件：源,通道,槽
  a2.sources = r2
  a2.channels = c2
  a2.sinks = k2
  
  
  # 1, 定义source相关信息，绑定流入的数据， 并制定输出的方向
  a2.sources.r2.type = avro
  a2.sources.r2.bind = localhost
  a2.sources.r2.port = 9902
  a2.sources.r2.channels = c2

  
  # 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
  a2.channels.c2.type = memory
  
  
  # 3, 定义sink相关的信息
  a2.sinks.k2.channel = c2
  a2.sinks.k2.type = logger
  ```

#### 2, 先启动a2
  > flume-ng agent -n a2 -c conf -f /usr/local/flume/conf/myConf/flume-hop-conf.properties -Dflume.root.logger=INFO,console



#### 3, 再启动a1
  > flume-ng agent -n a1 -c conf -f /usr/local/flume/conf/myConf/flume-hop-conf.properties -Dflume.root.logger=INFO,console

#### 4, ok了！

