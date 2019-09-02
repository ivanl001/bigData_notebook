## 1, zk单机版的安装



### 01, 解压zookeeper-3.4.9.tar.gz

* 放在/usr/local/目录下



### 02,配置zoo.cfg

* 更改配置文件，复制zoo_sample.cfg到一个新文件zoo.cfg中，并更改数据保存目录：

  ```properties
  dataDir=/tmp/zookeeper
  ```



### 03, 创建软链接



### 04, 配置/etc/profile

  ```shell
  export ZK_HOME=/usr/local/zookeeper
  export PATH=$PATH:$ZK_HOME/bin
  ```



### 05, 重新加载profile文件

```shell
source /etc/profile
```



### 06, 启动zk

* 启动后应该可以查看到有一个2181端口正在被监听

```shell
zkServer.sh start
```



### 07, 查看zk的状态

```shell
zkServer.sh status
```



### 09, 远程连接

```shell
zkCli.sh -server centos01:2181
```





## 2, zk分布式部署



### 01, 准备三台主机

* 我们这里用slave01, slave02, slave03



### 02，配置zoo.cfg

* 在zoo.cfg中添加服务器信息，其中，server.后面的数字是dataDir文件夹下的myid文件的内容，通过这个文件内容来识别不同的zookeeper

  ```properties
  dataDir=/data/zookeeper
  ```

server.1=slave01:2888:3888
server.2=slave02:2888:3888
server.3=slave03:2888:3888
  ```



### 03，配置myid

* 因为02中配置的myid文件的内容，所以需要在slave01, slave02, slave03三台服务器的/data/zookeeper文件夹下建立自己的myid文件，内容分别是上面对应的三个数值



### 04, 配置log4j.properties

* 顺便更改一下日志保存目录，也就是log4j.properties

  ```java
zookeeper.log.dir=/data/zookeeper/logs/
zookeeper.tracelog.dir=/data/zookeeper/logs/
  ```



### 05，启动

```shell
zkServer.sh start
```



### 06, 查看状态

* 查看每台服务器的状态,第一台启动的时候因为不够半数，所以既不是leader，也不是foller，全部启动后就ok了哈

```shell
zkServer.sh status
```



## 3, zk架构

```java
zk架构
------------------
	1.Client
		从server获取信息，周期性发送数据给server，表示自己还活着。
		client连接时，server回传ack信息。
		如果client没有收到reponse，自动重定向到另一个server.

	2.Server
		zk集群中的一员，向client提供所有service，回传ack信息给client，表示自己还活着。

	3.ensemble，也就是zk集群
		一组服务器。
		最小节点数是3.

	4.Leader
		如果连接的节点失败，自定恢复，zk服务启动时，完成leader选举。

	5.Follower
		追寻leader指令的节点。
		
znode
------------------
	zk中的节点，维护了stat，由Version number, Action control list (ACL), Timestamp,Data length.构成.
	data version		//数据写入的过程变化

	ACL					//action control list,这个好像是控制权限控制
  
  
节点类型
-----------------
	1.持久节点
		client结束，还存在。
		
	2.临时节点
		在client活动时有效，断开自动删除。临时节点不能有子节点。
		leader推选是使用。
		大概意思是leader推选的时候创建临时节点，如果这个Leader不幸dead了, 那么这个节点也就会被删除，其他服务器能监听到这个节点的删除，就能重新推选新的leader了。推选的规则在后面再介绍

	3.序列节点
		在节点名之后附加10个数字，主要用于同步和锁.
		
		
Session
--------------------
	Session中的请求以FIFO执行，一旦client连接到server，session就建立了。sessionid分配client.

	client以固定间隔向server发送心跳，表示session是valid的，zk集群如果在超时时候，没有收到心跳，
	判定为client挂了，与此同时，临时节点被删除。

Watches
-------------------
	观察。
	client能够通过watch机制在数据发生变化时收到通知。
	client可以在read 节点时设置观察者。watch机制会发送通知给注册的客户端。
	观察模式只触发一次。
	session过期，watch机制删除了。


zk工作流程
----------------
	zk集群启动后，client连接到其中的一个节点，这个节点可以leader，也可以follower。
	连通后，node分配一个id给client，发送ack信息给client。
	如果客户端没有收到ack，连接到另一个节点。
	client周期性发送心跳信息给节点保证连接不会丢失。


	如果client读取数据，发送请求给node，node读取自己数据库，返回节点数据给client.

	如果client存储数据，将路径和数据发送给server，server转发给leader。
	leader再补发请求给所有follower。只有大多数(超过半数)节点成功响应，则
	写操作成功。

```