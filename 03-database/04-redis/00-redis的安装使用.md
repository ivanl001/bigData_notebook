## 1, 下载

* https://redis.io/



## 2, 安装

```shell
# 解压
tar -zxvf redis-5.0.5.tar.gz -C /usr/local/

# 安装编译器
yum install gcc

# 进入根目录
cd /usr/local/redis-5.0.5/

# 编译
make

# 好像不管用
vim /usr/local/redis-5.0.5/redis.conf
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
# 设置默认开启后台运行daemon模式
daemonize yes
# 允许远程连接
protected-mode no
# 下面这样要注视掉！！！
# bind 127.0.0.1



# 启动redis-server
# /usr/local/redis-5.0.5/src/redis-server 

# 这样的话，配置文件中的配置才会生效哦
/usr/local/redis-5.0.5/src/redis-server /usr/local/redis-5.0.5/redis.conf 



nohup /usr/local/redis-5.0.5/src/redis-server /usr/local/redis-5.0.5/redis.conf > /dev/null 2>&1 & 
```



## 3, 连接

### 3.1, 内置客户端

```shell
/usr/local/redis-5.0.5/src/redis-cli


redis> set foo bar
OK
redis> get foo
"bar"

redis> shutdown


# 获取数据库，一共有16个数据库，分别是0-15
config get databases

# 使用3号库
select 3

# 查询所有key
keys *

# 
HGETALL 'foo'


```



## 4, 基本使用

```shell
# 查询所有key
keys *

LPUSH userId:4867 231449:3.0

LRANGE userId:4867 0 -1

del userId:4867
```

