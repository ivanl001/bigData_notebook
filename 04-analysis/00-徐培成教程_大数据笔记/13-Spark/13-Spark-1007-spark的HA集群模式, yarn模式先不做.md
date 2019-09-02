* 01，配置spark-env.sh，新增zk的ha功能，具体如下, 注意分发到每个节点上
  
> export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=slave01:2181,slave02:2181,slave03:2181 -Dspark.deploy.zookeeper.dir=/spark"

* 02, 关闭所有的spark节点，并重新启动
  
> ./start-all.sh

* 03, 查看主节点的状态
  
> http://master:8080/

* 04, 去备用节点上启动spark的master节点
  
> ./start-master.sh  # 这里是在master01上启动的

* 05, 查看备用节点的状态
  
> http://master01:8080/

* 06, 杀死主master节点，观察备用节点是否可以切换, 注意：这个过程大概要一分钟左右，需要耐心等待一下

* 07, 搞定！！！

