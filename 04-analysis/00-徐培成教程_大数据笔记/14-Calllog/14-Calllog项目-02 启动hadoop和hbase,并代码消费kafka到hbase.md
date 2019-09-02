> 4, 启动hadoop

> 5, 启动hbase

> 6, 写代码处理kafka主题并保存到->hbase->hdfs

> 7, 部署注意

## 4, 启动hadoop
  > start-all.sh
  
## 5, hbase
* 01，首先启动hbase
  > start-hbase.sh

* 02, 创建hbase表方便后续存储
  > create 'ns3:calllogs', 'f1', 'f2'

## 6, 写代码把kafka主题中消费的数据保存到hbase中，具体如下

*我们这里有四个类，两个配置文件:分别是:*
> KafkaToHbaseApp   #这个是主类
> IMHbaseService    #这个是服务层
> IMHbaseOperator   #这个相当于是dao层,保存到hbase中
> IMPropertiesUtils #这个相当于是dao层
> hbase-site.xml    #hbase的配置文件，通过这个文件连接hbase数据库
> kafka&hbase.properties   #自定义的一些属性

```java
package im.ivanl001.CallLogKafkaConsumer;

import java.io.InputStream;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Properties;

import kafka.consumer.Consumer;
import kafka.consumer.ConsumerConfig;
import kafka.consumer.ConsumerIterator;
import kafka.consumer.KafkaStream;
import kafka.javaapi.consumer.ConsumerConnector;

/**
 * #author      : ivanl001
 * #creator     : 2018-12-14 19:13
 * #description : 消费kafka的主题，然后保存到hbase中
 **/
public class KafkaToHbaseApp {

    public static void main(String[] args) throws Exception {

        /*//加载kafka的配置文件到输入流
        InputStream inputStream = ClassLoader.getSystemResourceAsStream("kafka&hbase.properties");
        //读取输入流到properties文件中
        Properties props = new Properties();
        props.load(inputStream);*/

        /*props.put("zookeeper.connect", "slave02:2181");
        props.put("group.id", "testgroup");
        props.put("zookeeper.session.timeout.ms", "50000");//这个时间设置500好像不行，太短了？？？
        props.put("zookeeper.sync.time.ms", "250");
        props.put("auto.commit.interval.ms", "1000");*/



        //String topic = props.getProperty("topic");
        //更新以下，直接用封装的工具取出指定的字段value
        String topic = IMPropertiesUtils.getProperty("topic");

        ConsumerConfig consumerConfig = new ConsumerConfig(IMPropertiesUtils.properties);
        ConsumerConnector consumer = Consumer.createJavaConsumerConnector(consumerConfig);

        //这里做一个map，可以同时获取多个主题到内容，我们这里就一个主题就可以了，所以就put一次即可
        Map<String, Integer> topicCountMap = new HashMap<String, Integer>();
        topicCountMap.put(topic, new Integer(1));
        //这里获取不同主题的内容，map中的key是主题，list是主题内容的列表
        Map<String, List<KafkaStream<byte[], byte[]>>> consumerStreams = consumer.createMessageStreams(topicCountMap);
        //这里获取我们所需要的主题的内容
        List<KafkaStream<byte[], byte[]>> streams = consumerStreams.get(topic);

        IMHbaseService imHbaseService = new IMHbaseService();

        //这里做处理即可
        for (final KafkaStream stream : streams) {
            ConsumerIterator<byte[], byte[]> consumerIte = stream.iterator();

            while (consumerIte.hasNext()) {
                String msg = new String(consumerIte.next().message());
                System.out.println("Message from Single Topic :: " + msg);
                imHbaseService.putLogsToHbaseOperator(msg);
            }
        }

        if (consumer != null){
            consumer.shutdown();
        }
    }
}
```

```java
package im.ivanl001.CallLogKafkaConsumer;

import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.util.Bytes;

import java.text.DecimalFormat;

/**
 * #author      : ivanl001
 * #creator     : 2018-12-15 11:37
 * #description :
 **/
public class IMHbaseService {

    private IMHbaseOperator imHbaseOperator = new IMHbaseOperator();
    private DecimalFormat decimalFormat = new DecimalFormat("00");
    
    public void putLogsToHbaseOperator(String logs) {

        System.out.println("IMHbaseService---putLogsToHbaseOperator---" + logs);

        //15810092493,13520404983,20180513140001,36
        //caller,calllee,time,duration
        String[] splitLogs = logs.split(",");
        if (splitLogs.length != 4) {
            return;
        }

        String caller = splitLogs[0];
        String callee = splitLogs[1];
        String time = splitLogs[2];
        String durationStr = splitLogs[3];

        //我们这里主叫log和被叫log直接一起存过去就好了， 就不用协处理器进行处理了
        //1,首先获取分区号
        String regionNo = getHashcode(caller, time);

        //2,获取主叫号码,已经有了，就是caller
        //3,获取当前月份
        String currentMonth = time.substring(0,6);
        //4,先处理主叫,所以
        String active = "0";
        //5,获取被叫号码,已经有了，就是callee

        //开始组建主叫rowkey,并进行存储
        String rowKey = regionNo + "," + caller + "," + currentMonth + "," + active + "," + callee + "," + durationStr;
        Put positivePut = new Put(rowKey.getBytes());
        positivePut.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("caller"), Bytes.toBytes(caller));
        positivePut.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("callee"), Bytes.toBytes(callee));
        positivePut.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("time"), Bytes.toBytes(time));
        positivePut.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("duration"), Bytes.toBytes(durationStr));
        imHbaseOperator.put(positivePut);

        //开始组建被叫rowkey,并进行存储
        String regionNo01 = getHashcode(callee, time);
        String active01 = "1";
        String rowKey01 = regionNo01 + "," + callee + "," + currentMonth + "," + active01 + "," + caller + "," + durationStr;
        Put negativePut = new Put(rowKey01.getBytes());
        negativePut.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("caller"), Bytes.toBytes(caller));
        negativePut.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("callee"), Bytes.toBytes(callee));
        negativePut.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("time"), Bytes.toBytes(time));
        negativePut.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("duration"), Bytes.toBytes(durationStr));
        imHbaseOperator.put(negativePut);
    }

    private String getHashcode(String caller ,String callTime){
        int len = caller.length();
        //取出后四位电话号码
        String last4Code = caller.substring(len - 4);
        //取出时间单位,年份和月份.
        String mon = callTime.substring(0,6);
        //这里是通过亦或的方式，这里不是很明白这里的和之前的hashcode的区别，应该都可以,我们这里假设分有100个区
        int hashcode = (Integer.parseInt(mon) ^ Integer.parseInt(last4Code)) % 100 ;
        return decimalFormat.format(hashcode);
    }
}
```

```java
package im.ivanl001.CallLogKafkaConsumer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Table;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2018-12-14 20:12
 * #description : 封装hbase的基本功能的一个工具类
 **/
public class IMHbaseOperator {

    private Table table;

    IMHbaseOperator(){
        Connection connection = null;
        try {
            Configuration configuration = HBaseConfiguration.create();
            connection = ConnectionFactory.createConnection(configuration);
            table = connection.getTable(TableName.valueOf(IMPropertiesUtils.getProperty("table.name")));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //put操作,这里做的比较简单，给我一个put对象，我就给你放进去即可
    public void put(Put put) {
        try {
            table.put(put);
            System.out.println("保存成功！！！");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
package im.ivanl001.CallLogKafkaConsumer;

import java.io.InputStream;
import java.util.Properties;

/**
 * #author      : ivanl001
 * #creator     : 2018-12-14 20:14
 * #description : 读取properties文件的工具类
 **/
public class IMPropertiesUtils {

    public static Properties properties;

    //本来想着可以传个文件名字进来再加载，想想好像性能也不好，而且配置文件也不多，所以就直接静态加载，后面如果再有其他的，可以重新建个新类即可
    static {
        try {
            //加载kafka的配置文件到输入流
            InputStream inputStream = ClassLoader.getSystemResourceAsStream("kafka&hbase.properties");
            properties = new Properties();
            //读取输入流到properties文件中
            properties.load(inputStream);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static String getProperty(String key) {
        return properties.getProperty(key);
    }
}
```


```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>

	<property>
	    <name>hbase.cluster.distributed</name>
	    <value>true</value>
	</property>

	<!--指定hbase数据在hdfs上的存放路径-->
	<property>
	    <name>hbase.rootdir</name>
	    <value>hdfs://master:8020/hbase</value>
	</property>

	<!-- 配置zk地址 -->
	<property>
	    <name>hbase.zookeeper.quorum</name>
	    <value>slave01:2181,slave02:2181,slave03:2181</value>
	</property>
	<!-- zk的本地目录 -->
	<property>
	    <name>hbase.zookeeper.property.dataDir</name>
	    <value>/data/zookeeper</value>
	</property>

</configuration>

```

```shell
zookeeper.connect=slave01:2181,slave02:2181,slave02:2181
group.id=hbase_group
zookeeper.session.timeout.ms=500
zookeeper.sync.time.ms=250
auto.commit.interval.ms=1000

auto.offset.reset=smallestc
#
topic=calllogs

# 这个是给hbase用的
table.name=ns3:calllogs
#
partition.number=100
#
caller.flag=0
#
hashcode.pattern=00
```

## 7，上述代码部署注意

* 01，打成jar包没啥好说的
* 
* 02，使用mvn命令，下载工件的所有依赖软件包，方式如下:
*在放有pom.xml的文件夹下建立一个lib文件夹，然后在pom.xml文件位置执行如下命令，会自动把所有依赖jar包下载到刚才的lib文件夹中去，注意修改其中的groupid什么的跟pom.xml中的一致哈*
  > mvn -DoutputDirectory=./lib -DgroupId=im.ivanl001 -DartifactId=14_CallLogKafkaConsumer -Dversion=1.0-SNAPSHOT dependency:copy-dependencies

* 03, 配置文件最好放在外面，和01中打好的jar包放在同一位置，方便修改，不然每次修改都需要重新打jar包太麻烦,方式如下：
  > 在lib同一目录下建立一个conf文件夹，把配置文件放进去即可（代码中有的话应该会覆盖吧，后面测一下），执行代码的时候加上conf即可，看后面的shell脚本

* 04，执行的时候：
  > java -cp CallLog_Kafka_Consumer.jar:lib/ im.ivanl001.CallLogKafkaConsumer.KafkaToHbaseApp # 实际这个并不行，所以还是写上每个jar包的具体位置，如下：

  * 类似这样的，但是太长了，所以我这里就不写了，直接写成一个shell命令中去了
  > java -cp CallLog_Kafka_Consumer.jar:lib/hadoop-common-2.5.1.jar im.ivanl001.CallLogKafkaConsumer.KafkaToHbaseApp


* 05, shell脚本内容如下：
*shell脚本的名字是：CallLog_kafka_consumer.sh* 
```shell
#! /bin/bash
# ./conf里面是配置文件
java -cp ./conf:CallLog_Kafka_Consumer.jar:./lib/commons-logging-1.2.jar:./lib/hadoop-auth-2.5.1.jar:./lib/jsr305-1.3.9.jar:./lib/activation-1.1.jar:./lib/commons-configuration-1.6.jar:./lib/commons-beanutils-1.7.0.jar:./lib/xz-1.0.jar:./lib/hbase-common-1.2.3.jar:./lib/commons-httpclient-3.1.jar:./lib/stax-api-1.0-2.jar:./lib/apacheds-i18n-2.0.0-M15.jar:./lib/hadoop-annotations-2.5.1.jar:./lib/slf4j-log4j12-1.7.21.jar:./lib/slf4j-api-1.6.1.jar:./lib/kafka-clients-0.10.0.1.jar:./lib/httpclient-4.2.5.jar:./lib/jline-0.9.94.jar:./lib/guava-12.0.1.jar:./lib/jaxb-api-2.2.2.jar:./lib/junit-4.12.jar:./lib/jopt-simple-4.9.jar:./lib/scala-parser-combinators_2.11-1.0.4.jar:./lib/hbase-client-1.2.3.jar:./lib/log4j-1.2.15.jar:./lib/hbase-annotations-1.2.3.jar:./lib/avro-1.7.4.jar:./lib/commons-cli-1.2.jar:./lib/commons-digester-1.8.jar:./lib/protobuf-java-2.5.0.jar:./lib/hadoop-yarn-common-2.5.1.jar:./lib/xmlenc-0.52.jar:./lib/jetty-util-6.1.26.jar:./lib/commons-codec-1.9.jar:./lib/jcodings-1.0.8.jar:./lib/commons-compress-1.4.1.jar:./lib/joni-2.1.2.jar:./lib/hadoop-yarn-api-2.5.1.jar:./lib/metrics-core-2.2.0.jar:./lib/commons-io-2.4.jar:./lib/jackson-core-asl-1.9.13.jar:./lib/scala-library-2.11.8.jar:./lib/mail-1.4.jar:./lib/hadoop-mapreduce-client-core-2.5.1.jar:./lib/commons-beanutils-core-1.8.0.jar:./lib/hbase-protocol-1.2.3.jar:./lib/netty-3.7.0.Final.jar:./lib/paranamer-2.3.jar:./lib/zookeeper-3.4.6.jar:./lib/commons-collections-3.2.2.jar:./lib/api-asn1-api-1.0.0-M20.jar:./lib/apacheds-kerberos-codec-2.0.0-M15.jar:./lib/hamcrest-core-1.3.jar:./lib/hadoop-common-2.5.1.jar:./lib/api-util-1.0.0-M20.jar:./lib/commons-net-3.1.jar:./lib/commons-lang-2.6.jar:./lib/lz4-1.3.0.jar:./lib/jsch-0.1.42.jar:./lib/snappy-java-1.1.2.6.jar:./lib/commons-el-1.0.jar:./lib/httpcore-4.2.4.jar:./lib/jackson-mapper-asl-1.9.13.jar:./lib/commons-math3-3.1.1.jar:./lib/netty-all-4.0.23.Final.jar:./lib/zkclient-0.8.jar:./lib/findbugs-annotations-1.3.9-1.jar:./lib/htrace-core-3.1.0-incubating.jar:./lib/kafka_2.11-0.10.0.1.jar:./hbase-site.xml:./CallLog_Kafka_Consumer.jar im.ivanl001.CallLogKafkaConsumer.KafkaToHbaseApp
```

* 06, 执行如下命令就可以启动程序了
*消费过的是不能重新消费的话，所以可以修改配置文件中的组信息，这样就可以重新从头消费了，是不是很开心，哈哈哈哈*
  
  > ./CallLog_kafka_consumer.sh