*以下内容都是在centos上安装使用*

## 1, mongodb的安装

### 1.1, 官网下载压缩包

* https://www.mongodb.com/download-center/community

* mongodb-linux-x86_64-rhel70-4.0.6.tgz



### 1.2,  解压，设置环境变量等

```shell
export MONGODB_HOME=/usr/local/mongodb
export PATH=$PATH:$MONGODB_HOME/bin
```



### 1.3, 启动

```shell
# 前台启动
mondod

# 后台启动
nohup mongod --bind_ip_all > /dev/null 2>&1 &
```





```shell
use admin

db.createUser({user:"ivanl001",pwd:"ivanl001",roles:["root"]}) 

db.createUser({user: "admin", pwd: "admin", roles: [{role: "readWrite", db: "admin"}]})

Successfully added user: { "user" : "admin", "roles" : [ "root" ] }




db.auth("admin","password")


4、MongoDB role类型

数据库用户角色

　　read:授予用户只读数据的权限

　　readWrite:授予用户读写数据的权限

数据库管理角色

dbAdmin:当前db中执行管理操作

dbOwner:当前DB中执行任意操作

userAdmin:当前DB中管理User
```





#### 2, mongodb客户端

##### 2.1, shell客户端进入

```shell
mongo
```



##### 2.2， 基本命令

* 显示数据库


```shell
show databases
show dbs
```



* 切换数据库,如果没有，会创建，但是只有数据插入才会真正的创建这个。已经存在，则切换到这个数据库上


```shell
use d1
show dbs   ---->   d1
```



* 插入


```shell
db.t1.insert({"name":"ivanl001", "age":10})
```



* 显示文集


```shell
show collections    ----->   t1
```



* 查询文档内容( 查询表中数据)


```shell
# db.是固定用法，表前面不是数据库的名字哦
db.t1.find()
db.t1.findOne()

db.rating.find({userId:4867})

# 升序
db.rating.find({userId:4867}).sort({timestamp:1})
```



* 更新文档, 只会更新最前面一条

```shell
db.t1.update({"name":"ivanl001"}, {$set:{"name":"ivanl001-modified"}})	
```





* 删除文档中数据(相当于表)


```shell
db.t1.drop()
```





* 删除数据库


```shell
use d1
db.dropDatabase()
show dbs
```





* 查看数据库级别帮助文档


```shell
db.help()
```





* 查看表级别的帮助文档


```shell
db.t1.help()
```





#### 3, Robot 3T

* netstat -nlp | grep mongo， 查看监听端口， 如果是127.0.0.1, 那么只允许本机连接，如果是0.0.0.0, 那么允许所有连接

* *默认监听端口是127.0.0.1， 这就导致不能远程连接mongodb的，如果需要用界面化工具连接，很简单，启动mongodb的时候加上mongod --bind_ip_all即可。如果只是使用mongod启动的时候，会有警告，说不推荐用root用户登录，当前不能远程连接等等。*

##### 3.1, 重启mongodb --bind_ip_all

> mongod --bind_ip_all

##### 3.2, 连接即可









