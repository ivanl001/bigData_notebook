> 如下的搭建都是在如下的虚拟机机器配置上进行的

## Storm的安装部署

### 服务器分布：3台：

| hostname | mem             | disk          | core |      |
| -------- | --------------- | ------------- | ---- | ---- |
| centos01 | 8G              | 50G           | 4    |      |
| centos02 | 3G              | 40G           | 3    |      |
| centos03 | 3G              | 40G           | 3    |      |
|          | free -h查看内存 | df -h查看磁盘 |      |      |

### 01, 下载，解压

- 软链，环境变量不再赘述

```shell
tar -zxvf apache-storm-1.1.3.tar.gz -C /usr/local/

ln -s /usr/local/apache-storm-1.1.3/ /usr/local/storm

# 要记得修改环境变量哦
```



### 02, 配置storm.yaml

```shell
vim /usr/local/storm/conf/storm.yaml 
```



```shell
# 这个是配置zk节点机器主机名
storm.zookeeper.servers:
  - "centos01"
  - "centos02"
  - "centos03"
  
# 这个是配置主节点机器主机名,我这里机器不够，就配置一台
nimbus.seeds: ["centos01","centos02"]


# 配置zk端口和存储目录
storm.zookeeper.port: 2181
storm.zookeeper.root: "/storm"

# 存储位置Nimbus dir
storm.local.dir: "/data/storm"
  
# 槽点端口
supervisor.slots.ports:
    - 6700
    - 6701
    - 6702
    - 6703
    
# UI
ui.host: 0.0.0.0
ui.port: 8080
```



### 03, 启动nimbus进程

> 记得先启动zk

* 分别在centos01,centos02上执行如下命令

```shell
storm nimbus &
```



### 04，启动supervisor

* 分别在centos01,centos02,centos03上执行如下命令

```shell
storm supervisor &
```



### 05, 启动ui

- 最后在nimbus上执行如下命令

```shell
storm ui &
```



### 06, 页面验证

- 如果配置正确，最终会有一个leader，三个supervisor，在ui页面上可以看的出来哈

* http://centos01:8080/index.html



### 07, 启动logviewer

* 如果想要在页面上查看某台节点的log, 执行如下命令

```shell
storm logviewer &
```



### 08, log服务器页面

* 比如说是查看在centos01上的log

* http://centos01:8000



## storm上运行jar包

```shell
storm jar Storm_test.jar im.ivanl001.bigData.Storm.A03_WordCount.WordCountApp
```

