## 0, 下载所需要的安装包

官网最新版本下载：https://prestodb.github.io/download.html

maven仓库指定版本下载：https://repo1.maven.org/maven2/com/facebook/presto/presto-server/



```shell
# presto server
presto-server-0.223.tar.gz

# presto命令行client
presto-cli-0.223-executable.jar 

presto-jdbc-0.223.jar
```





## 1, presto server的安装

部署参考：https://prestodb.github.io/docs/current/installation/deployment.html

### 1.1, 解压缩presto-server-0.223.tar.gz到指定目录

```shell
tar -zxvf presto-server-0.223.tar.gz -C /usr/local/
```

### 1.2, 配置环境变量，创建软链接

```shell
ln -s presto-server-0.223/ presto

vim /etc/profile

# 在profile中添加如下两行
export NODE_HOME=/usr/local/node
export PATH=$PATH:$NODE_HOME/bin

# 更新profile
source /etc/profile
```

### 1.3, 创建presto数据目录

```shell
mkdir /data/presto
```



下面配置参考：https://www.cnblogs.com/defineconst/p/11242565.html



### 1.4, 创建/etc目录，并配置如下几个配置文件

```shell
mkdir /usr/local/presto/etc
```

### 1.5, 配置文件1:node.properties

* node.properties

```properties
node.environment=production
# 这个需要在每台机器上不一样，比如说我的centos01上结尾是fa, centos02上结尾是fb，centos03上结尾是fc
node.id=ffffffff-ffff-ffff-ffff-fffffffffffa
node.data-dir=/data/presto
```

### 1.6, 配置文件2:jvm.config

* jvm.config

```properties
-server
-Xmx16G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError
```

### 1.7, 配置文件3: config.properties

`⚠️注意：这个在不同的机器上是需要不同的，如果想把这台机器作为coordinator，需要配置如下coordinator内容：,如果想作为worker需要配置内容和后面的那个work文件一样` 

我们这里centos01是作为coordinator，centos02，centos03作为worker，所以centos01上的配置文件内容和下面这个一样，02，03机器上的配置内容和后面那个一样

⚠️coordinator的配置文件内容：

```properties
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8090
query.max-memory=50GB
query.max-memory-per-node=1GB
query.max-total-memory-per-node=2GB
discovery-server.enabled=true
discovery.uri=http://centos01:8090
```

⚠️work的配置文件内容：

```properties
coordinator=false
http-server.http.port=8090
query.max-memory=50GB
query.max-memory-per-node=1GB
query.max-total-memory-per-node=2GB
discovery.uri=http://centos01:8090
```

### 1.8, 配置文件4: etc/catalog目录下：hive.properties和jmx.properties

* Create `etc/catalog/hive.properties` with the following contents to mount the `hive-hadoop2` connector as the `hive` catalog, replacing `example.net:9083` with the correct host and port for your Hive metastore Thrift service:
* ⚠️注意： hive.properties配置core-site.xml和hdfs-site.xml文件，否则会报nameservices1错误，hdfs的HA导致的哈，配置了之后就好了
* hive.properties文件如下：

```properties
connector.name=hive-hadoop2
hive.metastore.uri=thrift://centos01:9083,thrift://centos02:9083
hive.config.resources=/etc/hadoop/conf/core-site.xml,/etc/hadoop/conf/hdfs-site.xml
```

* jmx.properties文件如下：

```properties
connector.name=jmx
```



### 1.9, 启动presto

`⚠️注意：启动前需要先启动hive`

```shell
# 我们这里后台启动
/usr/local/presto-server/bin/launcher start

# 如果想要打印出日志信息到控制太，可以使用如下命令
/usr/local/presto-server/bin/launcher run
```



## 2, presto命令行client的安装

`⚠️注意：企业开发一般很少会直接单独使用命令行操作，主要还是使用可视化的client，这里只是做学习使用`

部署参考：https://prestodb.github.io/docs/current/installation/cli.html

```shell
Download presto-cli-0.223-executable.jar, 
rename it to presto, 
make it executable with chmod +x, 
then run it:

./presto --server centos01:8080 --catalog hive --schema default
```

### 2.1, 把下载的jar包重命名为presto

```shell
mv presto-cli-0.223-executable.jar presto
```

### 2.2, 给presto增加可执行权限

```shell
chmod +x presto
```

### 2.3, 把presto移动到/usr/loca/bin目录下方便执行

```shell
mv presto /usr/local/bin/
```

### 2.4, 执行

```shell
# 端口号就是上面server中配置的端口号
presto --server localhost:8090 --catalog hive --schema default
```



## 3, presto可视化client的安装

yanagishima官网：https://github.com/zhaolianchao/yanagishima



### 3.1, 下载yanagishima-18.0.zip

### 3.2, 解压缩

### 3.3, 修改配置文件：yanagishima.properties 

```shell
# 这个是ui页面的端口，修改后通过这个端口打开页面
jetty.port=8089

# 这个是服务器的端口，也就是上面1中服务器配置什么端口和host，这里就是什么端口和host
presto.coordinator.server.your-presto=http://centos01:8090
```

### 3.4, 启动

* 开启

```shell
nohup bin/yanagishima-start.sh >y.log 2>&1 &
```

* 关闭

```shell
bin/yanagishima-shutdown.sh
```

