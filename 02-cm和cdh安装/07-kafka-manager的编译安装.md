kafka-manager的具体使用参考如下：

https://www.jianshu.com/p/6a592d558812



## 一, 安装

### 1, 官方下载源码

kafka-manager 项目地址：https://github.com/yahoo/kafka-manager

最好下载release版本：https://github.com/yahoo/kafka-manager/releases



### 2, 在服务器上解压缩

```shell
tar -zxvf kafka-manager-2.0.0.2.tar.gz
```



### 3, 安装sbt，后面通过sbt进行编译

```shell
# 
curl https://bintray.com/sbt/rpm/rpm > bintray-sbt-rpm.repo
mv bintray-sbt-rpm.repo /etc/yum.repos.d/
yum install sbt -y
sbt -version
```



### 4, sbt编译

```shell
# 然后cd到kafka-manager的解压目录，通过sbt编译
./sbt clean dist
# 可能要等很久，编译好后会有一个zip包，解压缩好了
```

> 看到打印这个消息 Getting org.scala-sbt sbt 0.13.9  (this may take some time)... 就慢慢等吧，可以到~/.sbt/boot/update.log 查看sbt更新日志。sbt更新好，就开始下载各种jar包，
>
> 最后看到：Your package is ready in /usr/local/kafka-manager-2.0.0.2/target/universal/kafka-manager-2.0.0.2.zip  证明编译好了。



### 5, 安装编译好的kafka-manager

```shell
unzip /usr/local/kafka-manager-2.0.0.2.zip
```



### 6, 配置kafka-manager的zk启动即可

```shell
# 注意建立软连接哦
vim /usr/local/kafka-manager/conf/application.conf

# 好像修改这一个地方即可
kafka-manager.zkhosts="centos01:2181,centos02:2181,centos03:2181"
```



### 7, 启动kafka-manager

```shell
# 后台启动，也可以指定端口等
nohup /usr/local/kafka-manager/bin/kafka-manager -Dconfig.file=conf/application.conf -Dhttp.port=9999 &
```



### 8, 查看启动状态

http://centos01:9999



## 二, 配置kafka

具体的配置参考这个：https://www.jianshu.com/p/6a592d558812

![image-20190724180536665](assets/image-20190724180536665.png)