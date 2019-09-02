*在这之前请先参考“03-Hadoop-0901-机架感知和HA”的HA配置*



## 1，hdfs的自动容灾配置

### 01, 关闭所有的jps进程

```shell
stop-all.sh
```



### 02, 配置hdfs-site.xml

* 新增如下配置:

  ```xml
  <property>
	  <name>dfs.ha.automatic-failover.enabled</name>
		<value>true</value>
	</property>
  ```


### 03, 配置core-site.xml

* 新增如下配置

  ```xml
  <property>
		<name>ha.zookeeper.quorum</name>
		<value>slave01:2181,slave02:2181,slave03:2181</value>
	</property>
  ```



### 04, 同步所有的上面两个文件

### 05，关闭zookeeper

* 最好清理一下version，log等没用的文件，然后重新开启zookeeper

### 06, 初始化zk

* 登陆其中一个nn主机, 执行如下命令

```shell
hdfs zkfc -formatZK
```



### 07，启动集群

* 命令如下

```shell
start-dfs.sh
```





## 2, rm(resourcemanager)的自动容灾

### 01，配置yarn-site.xml

* 添加如下属性

  ```xml
<property>
  <name>yarn.resourcemanager.ha.enabled</name>
  <value>true</value>
</property>
<property>
  <name>yarn.resourcemanager.cluster-id</name>
  <value>cluster1</value>
</property>
<property>
  <name>yarn.resourcemanager.ha.rm-ids</name>
  <value>rm1,rm2</value>
</property>
<property>
  <name>yarn.resourcemanager.hostname.rm1</name>
  <value>master</value>
</property>
<property>
  <name>yarn.resourcemanager.hostname.rm2</name>
  <value>master01</value>
</property>
<property>
  <name>yarn.resourcemanager.webapp.address.rm1</name>
  <value>master:8088</value>
</property>
<property>
  <name>yarn.resourcemanager.webapp.address.rm2</name>
  <value>master01:8088</value>
</property>
<property>
  <name>yarn.resourcemanager.zk-address</name>
  <value>slave01:2181,slave02:2181,slave03:2181</value>
</property>
  ```
### 02.启动yarn集群

```shell
start-yarn.sh
```



### 03.hadoop没有启动两个resourcemanager,需要手动启动另外一个

```shell
yarn-daemon.sh start resourcemanager
```



### 04, 使用管理命令
```shell
//查看状态
yarn rmadmin -getServiceState rm1

//切换状态到standby
yarn rmadmin -transitionToStandby rm1
```



### 5.查看webui

* http://master:8088
* http://master01:8088



### 6.做容灾模拟

```shell
# 通过如下命令杀死需要的那个pid即可
kill -9 pid
```







