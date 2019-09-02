> 如下的搭建都是在如下的虚拟机机器配置上进行的

##### 服务器分布：3台：

| hostname | mem             | disk          | core |      |
| -------- | --------------- | ------------- | ---- | ---- |
| centos01 | 8G              | 50G           | 4    |      |
| centos02 | 3G              | 40G           | 3    |      |
| centos03 | 3G              | 40G           | 3    |      |
|          | free -h查看内存 | df -h查看磁盘 |      |      |

##1, local

### 1.1, 解压

```shell
tar -zxvf spark-2.1.0-bin-without-hadoop.tgz -C /usr/local/
```

### 1.2, 启动

```shell
./spark-shell --master local
```

### 1.3, 页面验证

* http://192.168.147.101:4040



### 1.4, 提交作业

#### 1.4.1, 代码中配置

```java
config.setMaster("local")
```

#### 1.4.2, 提交

```shell
spark2-submit --master local --name IMWordCount  --class im.ivanl001.bigData.Spark.A01_WordCount.a01_WordCount_App01_local Spark_Test.jar /user/ivanl001/ivanl001.txt
```



----

如果部署的是集群版本的，有如下几种模式：

![image-20190817075754196](assets/image-20190817075754196.png)

## 2, standalone

> 这个属于独立模式，spark不依赖其他(如yarn)单独运行集群

### 2.1, 配置slave文件

```shell
vim /usr/local/spark/conf/slaves

# 增加如下内容
centos01
centos02
centos03
```



### 2.2，设置jdk

```shell
/sbin/spark-config.sh

# 增加如下一句
export JAVA_HOME=/usr/local/jdk
```



### 2.3, 启动spark

```shell
/usr/local/spark-2.1.0-bin-hadoop2.6/sbin/start-all.sh 
```



### 2.4, 页面验证

> 页面上会有如下信息：Spark Master at spark://master:7077，提交分布式作业的时候需要用到spark://master:7077

* http://centos01:8080/



### 2.5, 关闭spark

```shell
/usr/local/spark-2.1.0-bin-hadoop2.6/sbin/stop-all.sh
```



### 2.6, 提交作业

#### 2.6.1, 代码中配置

```java
//spark://centos01:7077可以在页面中找到
val conf = new SparkConf()
conf.setMaster("spark://centos01:7077")
```

#### 2.6.2, 提交

```shell
spark-submit --master spark://master:7077 --name IMWordCount  --class im.ivanl001.bigData.Spark.A01_WordCount.a01_WordCount_App02_standalone Spark_Test.jar hdfs://centos01:8020/user/ivanl001/ivanl001.txt
```





## 3, HA(standalone)

* 其实就是driver的HA模式

### 3.1, 配置spark-env.sh

```shell
vim spark-env.sh

# 增加如下zk内容
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=centos01:2181,centos02:2181,centos03:2181 -Dspark.deploy.zookeeper.dir=/spark"

imrsync.sh spark-env.sh
```



### 3.2, 启动主节点

```shell
# 这个在centos01上启动
/usr/local/spark-2.1.0-bin-hadoop2.6/sbin/start-all.sh 
```



### 3.3, 主节点页面验证

* http://centos01:8080/



![image-20190511174839994](assets/image-20190511174839994.png)

### 3.4, 备用节点上启动spark的master节点

```shell
/usr/local/spark-2.1.0-bin-hadoop2.6/sbin/start-master.sh  # 这里是在centos02上启动的
```



### 3.5, 查看备用节点的状态

* http://centos02:8080/



![image-20190511174905837](assets/image-20190511174905837.png)

### 3.6, 手动试用切换

>  杀死主master节点，观察备用节点是否可以切换, 注意：这个过程大概要一分钟左右，需要耐心等待一下



- ![image-20190511175001914](assets/image-20190511175001914.png)
- ![image-20190511175051555](assets/image-20190511175051555.png)

### 3.7, 搞定！



### 3.8, 提交作业

* 这里其实和非ha分布式的是一样的

#### 3.8.1, 代码中配置

```java
//spark://centos01:7077可以在页面中找到
val conf = new SparkConf()
conf.setMaster("spark://centos01:7077")
```

#### 3.8.2, 提交

```shell
spark-submit --master spark://master:7077 --name IMWordCount  --class im.ivanl001.bigData.Spark.A01_WordCount.a01_WordCount_App02_standalone Spark_Test.jar hdfs://centos01:8020/user/ivanl001/ivanl001.txt
```



## 4, CDH-spark-on-yarn

### 4.1, 安装

* 参考“02-cm和cdh安装/04-cdh安装其他02-spark2.1.0.md”文件

### 4.2, 提交应用

#### 4.2.1, 代码中配置

```java
//代码中设置Master参数
val conf = new SparkConf()
config.setMaster("yarn")
```

#### 4.2.2, 提交

```shell
# 因为我设置了高可用，所以就不具体指定是那台服务器，默认知道是hdfs协议的
spark2-submit --master yarn --name IMWordCount  --class im.ivanl001.bigData.Spark.A01_WordCount.a01_WordCount_App03_yarn Spark_Test.jar /user/ivanl001/ivanl001.txt
```

```shell
# 如果并发度比较高，默认会占用大量资源，比如我这个代码里面开启了4个并发度，就直接报错
# 所以最后只能设置参数--executor-memory 1G --total-executor-cores 2限制资源
spark2-submit --master yarn --name noDateLean --executor-memory 1G --total-executor-cores 2   --class im.ivanl001.bigData.Spark.A04_DataLean.A0401_WordCountApp_NoDataLean_on_yarn Spark_Test.jar /user/ivanl001/00-bigData/input/zhang02.txt /user/root/nodataLeanOut03/
```

```java
spark2-submit --master yarn --name broadcast --executor-memory 1G --total-executor-cores 2   --class im.ivanl001.bigData.Spark.A07_PersistAndCacheAndBroadcast.A0703_Broadcast_on_yarn Spark_Test.jar
```

https://www.jianshu.com/p/b3d87df219e9

```shell
# 最小450M
spark2-submit --master yarn --name socketStream_demo --executor-memory 1G --executor-cores 6 --num-executors 3 --class im.ivanl001.bigData.Spark.A09_SparkStream.A0901_SparkStreamDemo_on_yarn Spark_Test.jar


spark2-submit --master yarn --name socketStream_demo --class im.ivanl001.bigData.Spark.A09_SparkStream.A0901_SparkStreamDemo_on_yarn Spark_Test.jar



$SPARK_HOME/bin/spark-submit --master yarn --deploy-mode cluster --class you.class.Name --executor-memory 1g --executor-cores 1 --num-executors 8 --driver-memory 2g  /you/jar.jar  
 ———————————————— 
版权声明：本文为CSDN博主「AI_skynet」的原创文章，遵循CC 4.0 by-sa版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/xinganshenguang/article/details/53436196
```



## 5, spark的两种作业提交模式

### 5.1, client模式

* 提交作业的机器充当driver，接受打印信息等操作
* executor返回信息给driver
* 比如直接开一个spark2-shell的时候就是这种模式
* 默认是client模式



### 5.2, cluster模式

* spark选择worker中的其中一个充当driver，会找空闲的充当
* 这样分担了提交机器的压力
* 直接提交spark作业的时候是这种模式



