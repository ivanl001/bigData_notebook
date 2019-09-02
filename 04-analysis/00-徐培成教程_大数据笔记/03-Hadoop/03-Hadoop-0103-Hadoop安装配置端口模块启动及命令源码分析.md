```
hadoop
----------------
	开源软件，可靠的、分布式、可伸缩的。

去IOE
-------------
	IBM			//ibm小型机.
	Oracle		//oracle数据库服务器 RAC
	EMC			//EMC共享存储设备。
	
大数据解决了两个问题
----------------------
	1.存储
		分布式存储
	2.计算
		分布式计算
```



Hadoop四个模块：

 * 1, common
 * 2, hdfs
 * 3, mapreduce
 * 4, yarn 



## 1, Hadoop基本安装

* cdh中使用的是2.6.0

### 1.1，解压缩

```shell
tar -zxvf hadoop-2.7.5.tar.gz
```

### 1.2, 放到安装目录

```shell
mv hadoop-2.7.5 /usr/local/ 
```

* 1.1, 1.2可以合并成一条命令

```shell
# 这样会直接解压到指定目录
tar -zxvf hadoop-2.7.5.tar.gz -C /usr/loca/
```



### 1.3, 创建软链

```shell
ln -s /usr/local/hadoop-2.7.5/ /usr/local/hadoop 
```



### 1.4, 添加环境变量

```shell
vim /etc/profile 

export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

### 1.5，验证版本

```shell
[root@master local]# hadoop version
Hadoop 2.7.5
Subversion https://shv@git-wip-us.apache.org/repos/asf/hadoop.git -r 18065c2b6806ed4aa6a3187d77cbe21bb3dba075
Compiled by kshvachk on 2017-12-16T01:06Z
Compiled with protoc 2.5.0
From source with checksum 9f118f95f47043332d51891e37f736e9
This command was run using /usr/local/hadoop-2.7.5/share/hadoop/common/hadoop-common-2.7.5.jar
```

## 2，Hadoop不同模式的配置

### 2.1， standalone(local)

*单机版模式*

> 默认安装好了之后就是单机版，不需要更改任何文件

### 2.2，pseudodistributed  mode

*伪分布模式, 配置下面四个文件*

#### 01，core-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost/</value>
    </property>
</configuration>
```

#### 02，hdfs-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

#### 03，mapred-site.xml

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

#### 04，yarn-site.xml

```xml
<?xml version="1.0"?>
<configuration>
<!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

#### 启动hadoop

#### 05, 格式化

 启动前先进行格式化，只有第一次启动前格式化，格式化会删除已有hadoop的数据

```shell
hadoop namenode -format
```



#### 06，启动hadoop

* startall.sh，如果报错jdk找不到，到hadoop-env.sh中修改jdk的路径为绝对路径重新启动应该就好了

```shell
start-all.sh
```



#### 07, jps查看启动状态

```shell
jps

jps如果显示出五个进程说明ok
namenode 
datenode 
nodemanager 
secondarynamenode 
resourcemanager
```



#### 08, 页面验证：端口50070

http://centos01:50070



`注意：需要先关闭禁用防火墙`

> systemctl status firewalld.service
> systemctl enable firewalld.service
> systemctl disable firewalld.service
> systemctl start firewalld.service
> systemctl stop firewalld.service


### 2.3, fully distributed mode

*完全分布式模式*

#### 00, 先更改slaves文件

> vim /usr/local/hadoop/etc/slaves

```shell
# 添加datanode的节点
192.168.217.121
192.168.217.122
192.168.217.123
192.168.217.124
```
* 为了方便namenode节点和datanode节点相互通信，建议修改一下namenode机器的hosts文件，如下：
```shell
127.0.0.1   localhost

# 这一台是namenode节点
192.168.217.120 master

# 下面三台是datanode节点
192.168.217.121 slave01
192.168.217.122 slave02
192.168.217.123 slave03
```

#### 01, core-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>fs.defaultFS</name>
        <!-- 这里是你的namenode的地址 -->
        <value>hdfs://192.168.217.120/</value>
    </property>
</configuration>
```

#### 02, hdfs-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>dfs.replication</name>
        <!-- 这里是你的datanode的数量, 准确的说不是datanode的数据，而是每个文件需要保存几个副本，如果是3，就会保存3份 -->
        <value>3</value>
    </property>
</configuration>
```

#### 03, mapred-site.xml

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

#### 04, yarn-site.xml

```xml
<?xml version="1.0"?>

<configuration>
<!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <!-- 这里是你的资源管理器的地址，其实应该就是namenode节点所在服务器ip -->
        <value>192.168.217.120</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```




## 3, 拓展: hadoop的端口

```shell
NameNode             http port 50070    rpc 8020
DataNode             http port 50075    rpc 50010
SecondaryNameNode    http port 50090

NodeManager          
ResourceManager                8088
```



## 4, hadoop的四个模块

```shell
common
hdfs    //namenode+datanode+secondarynamenode
mapred  
yarn    //resourcemanager+nodemanager
```



## 5, hadoop启动脚本

```shell
# 第一种方式
start-all.sh
# stop-all.sh

# 第二种方式
start-dfs.sh //这个会启动namenode, datanode, secondarynamenode
start-yarn.sh //这个会启动resourcemanager, nodemanager
```



## 6， 源码分析

```shell
脚本分析
-------------------
	sbin/start-all.sh
	--------------
		libexec/hadoop-config.sh
		start-dfs.sh
		start-yarn.sh

	sbin/start-dfs.sh
	--------------
		libexec/hadoop-config.sh
		sbin/hadoop-daemons.sh --config .. --hostname .. start namenode ...
		sbin/hadoop-daemons.sh --config .. --hostname .. start datanode ...
		sbin/hadoop-daemons.sh --config .. --hostname .. start sescondarynamenode ...
		sbin/hadoop-daemons.sh --config .. --hostname .. start zkfc ...			//

	sbin/start-yarn.sh
	--------------	
		libexec/yarn-config.sh
		bin/yarn-daemon.sh start resourcemanager
		bin/yarn-daemons.sh start nodemanager
	

	sbin/hadoop-daemons.sh
	----------------------
		libexec/hadoop-config.sh

		slaves

		hadoop-daemon.sh

	sbin/hadoop-daemon.sh
	-----------------------
		libexec/hadoop-config.sh
		bin/hdfs ....
	

	sbin/yarn-daemon.sh
	-----------------------
		libexec/yarn-config.sh
		bin/yarn

```