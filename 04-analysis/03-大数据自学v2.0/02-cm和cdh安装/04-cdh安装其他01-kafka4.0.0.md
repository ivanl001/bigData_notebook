##### 1, kafka的安装

######  1.1, 先是安装parcel

* 如下图，找到kafka，下载后分配，然后再激活

![image-20190510054325790](assets/image-20190510054325790.png)



###### 1.2, 然后先是到配置目录中修改配置文件

> vim /opt/cloudera/parcels/KAFKA/etc/kafka/conf.dist/server.properties

```properties
# 这个要修改哈，没台机器不一样的哈
broker.id=0

# 打开端口，这行默认注释，打开这行
listeners=PLAINTEXT://:9092
# 更改log的目录
log.dirs=/data/kafka/kafka-logs
# 配置zk的集群服务器
zookeeper.connect=centos01:2181,centos02:2181,centos03:2181

port=9092
host.name=cenos01
```



###### 1.3, 配置

* 然后点击启动后，需要在配置页面配置如下几项：

![image-20190510054729581](assets/image-20190510054729581.png)

![image-20190510054837582](assets/image-20190510054837582.png)



###### 1.4， 保存更改，开启即可，成功

###### 1.5, 可能不明白的地方：

然后这里遇到的问题是填写Destination Brokers List和填写Source Brokers List 。

1. 填写Destination Brokers List 
若添加了Kafka MirrorMaker，则可填写其所在节点构成的列表,若未添加Kafka MirrorMaker，可填写任意服务器即可，如下：master:9092;salve1:9092;salve2:9092
2填写Source Brokers List 
填写Kafka Broker所在节点构成的列表（用逗号分隔），如下（我在所有节点部署了Kafka Broker）master:9092;salve1:9092;salve2:9092

![image-20190510052230573](assets/image-20190510052230573.png)

![image-20190510052242904](assets/image-20190510052242904.png)







