## 1, 日志的生成

> ⚠️注意：根据项目中logback.xml中的设置，日志文件被放置在了/tmp/logs/目录下

### 1.1, 日志生成系统代码

* 代码这里不再展示，参照地址如下：

* learning_code/04-analysis/02-BigData_v3/01-logcollector

### 1.2, 打包

* 使用带依赖的jar包，因为项目中使用了fastjson

* 01-log-collector-1.0-SNAPSHOT-jar-with-dependencies.jar

### 1.3, 运行生成日志

```shell
# 第一个参数表示两条数据之间的时间间隔，毫秒
# 第二个参数表示生成的条数，我这里生成10000条数据
java -jar /root/06-dataware/01-log-collector-1.0-SNAPSHOT-jar-with-dependencies.jar 0 10000
```



## 2, flume日志搜集

> ⚠️注意：根据不同的版本的flume可以使用不同的搜集策略。
>
> 我使用的是flume1.6.0版本，所以使用exec的策略演示，也可以用spooldir
>
> 如果是flume1.7.0，可以使用Taildir，可以进行断点续传
>
> 然后下面的两个拦截器是放在不

### 2.1, 清洗拦截器

* 主要是做简单的数据清洗判断，比如某个字段为空，则不搜集等等

```java
package com.atguigu.flume.interceptor;

import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.nio.charset.Charset;
import java.util.ArrayList;
import java.util.List;

/**
 * #author      : ivanl001
 * #creator     : 2019-08-03 17:35
 * #description :
 **/
public class LogETLInterceptor implements Interceptor{
    @Override
    public void initialize() {
    }

    @Override
    public Event intercept(Event event) {
        byte[] body = event.getBody();
        String msg = new String(body, Charset.forName("UTF-8"));
        if (msg.contains("start")) {
            if (msg.trim().startsWith("{")) {
                return event;
            }
        } else{
            if (!(msg.length() == 0)) {
                return event;
            }
        }
        return null;
    }

    @Override
    public List<Event> intercept(List<Event> events) {

        List<Event> events1 = new ArrayList<>();
        for (Event e : events) {
            Event intercept = intercept(e);
            if (intercept != null) {
                events1.add(intercept);
            }
        }
        return events1;
    }

    @Override
    public void close() {
    }

    public static class Builder implements Interceptor.Builder{
      
        @Override
        public Interceptor build() {
            return new LogETLInterceptor();
        }
      
        @Override
        public void configure(Context context) {

        }
    }
}
```

### 2.2, 类型判断拦截器

* 主要是通过日志中的内容，判断出不同的日志，通过event的header记录下来，然后在flume的sink中根据不同的header写入到不同的sink，也就是不同的kafka主题中去

```java
package com.atguigu.flume.interceptor;

import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.nio.charset.Charset;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

/**
 * #author      : ivanl001
 * #creator     : 2019-08-03 17:57
 * #description : 
 **/
public class LogTypeInterceptor implements Interceptor {
    @Override
    public void initialize() {

    }

    @Override
    public Event intercept(Event event) {
        String body = new String(event.getBody(), Charset.forName("UTF-8"));
        Map<String, String> headers = event.getHeaders();

        if (body.contains("start")){
            headers.put("topic", "topic_start");
        }else{
            headers.put("topic", "topic_event");
        }
        return event;
    }

    @Override
    public List<Event> intercept(List<Event> events) {
        List<Event> events1 = new ArrayList<>();
        for (Event e : events) {
            Event intercept = intercept(e);
            if (intercept != null) {
                events1.add(intercept);
            }
        }
        return events1;
    }

    @Override
    public void close() {

    }

    public static class Builder implements Interceptor.Builder{
      
        @Override
        public Interceptor build() {
            return new LogTypeInterceptor();
        }
      
        @Override
        public void configure(Context context) {

        }
    }
}
```

### 2.3, 拦截器项目打成jar包

* 使用不带依赖的即可，因为所有的依赖flume本身都有的

* 02-flume-interceptor-1.0-SNAPSHOT.jar

### 2.4, 把拦截器jar包放置到flume的lib目录

```shell
mv 02-flume-interceptor-1.0-SNAPSHOT.jar /opt/cloudera/parcels/CDH/lib/flume-ng/lib
imrsync.sh /opt/cloudera/parcels/CDH/lib/flume-ng/lib/02-flume-interceptor-1.0-SNAPSHOT.jar
```

### 2.5, 搜集flume的配置

```properties
# 0，定义组件
a1.sources=r1
a1.channels=c1 c2 
a1.sinks=k1 k2 

# 1，配置source， configure source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /tmp/logs/app-2019-08-05.log
# Whether to add a header storing the absolute path filename.
# 是否添加header
a1.sources.r1.fileHeader = true
a1.sources.r1.channels = c1 c2

# 2，配置拦截器，interceptor
a1.sources.r1.interceptors = i1 i2
a1.sources.r1.interceptors.i1.type = com.atguigu.flume.interceptor.LogETLInterceptor$Builder
a1.sources.r1.interceptors.i2.type = com.atguigu.flume.interceptor.LogTypeInterceptor$Builder

# 3，通过拦截器代码，配置通道选择器，selector
a1.sources.r1.selector.type = multiplexing
a1.sources.r1.selector.header = topic
a1.sources.r1.selector.mapping.topic_start = c1
a1.sources.r1.selector.mapping.topic_event = c2

# 4，配置通道，configure channel
a1.channels.c1.type = memory
a1.channels.c1.capacity=10000
a1.channels.c1.byteCapacityBufferPercentage=20

a1.channels.c2.type = memory
a1.channels.c2.capacity=10000
a1.channels.c2.byteCapacityBufferPercentage=20

# 5，配置sink，configure sink
# 5.1，第一个sink，进入一个kafka主题：topic_start中
a1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
a1.sinks.k1.kafka.topic = topic_start
a1.sinks.k1.kafka.bootstrap.servers = centos01:9092,centos02:9092,centos03:9092
a1.sinks.k1.kafka.flumeBatchSize = 2000
a1.sinks.k1.kafka.producer.acks = 1
a1.sinks.k1.channel = c1

# 5.2，第二个sink，进入另外一个kafka主题：topic_event中
a1.sinks.k2.type = org.apache.flume.sink.kafka.KafkaSink
a1.sinks.k2.kafka.topic = topic_event
a1.sinks.k2.kafka.bootstrap.servers = centos01:9092,centos02:9092,centos03:9092
a1.sinks.k2.kafka.flumeBatchSize = 2000
a1.sinks.k2.kafka.producer.acks = 1
a1.sinks.k2.channel = c2
```

### 2.6, 等待kafka启动ok了再启动flume即可

## 3, kafka及压力测试

* kafka没有太多需要配置的
* cdh安装后，根据提示修改Kafka的堆大小为256M
* Java Heap Size of Broker 256M

* 启动kafka即可
* 然后启动2步中的flume，当有日志生成的时候就会有消息进入kafka，可以用kafka-manager进行查看

### 3.1, 生产能力测试

```shell
# 生产能力测试命令如下：
/opt/cloudera/parcels/KAFKA/bin/kafka-producer-perf-test --topic test --record-size 100 --num-records 100000 --throughput 1000 --producer-props bootstrap.servers=centos01:9092,centos01:9092,centos01:9092
```

说明：record-size是一条信息有多大，单位是字节。num-records是总共发送多少条信息。throughput 是每秒多少条信息。

结果如下：

```java
19/08/03 19:12:08 INFO producer.KafkaProducer: [Producer clientId=producer-1] Closing the Kafka producer with timeoutMillis = 9223372036854775807 ms.
100000 records sent, 999.790044 records/sec (0.10 MB/sec), 2.72 ms avg latency, 475.00 ms max latency, 1 ms 50th, 2 ms 95th, 48 ms 99th, 381 ms 99.9th.

// 参数解析：本例中一共写入10w条消息，每秒向Kafka写入了0.10MB的数据，平均是1000条消息/秒，每次写入的平均延迟为2.72毫秒，最大的延迟为475毫秒。
```



### 3.2, 消费能力测试

```shell
# 消费能力测试如下
/opt/cloudera/parcels/KAFKA/bin/kafka-consumer-perf-test --broker-list  centos01:9092,centos02:9092,centos03:9092 --topic test --fetch-size 10000 --messages 10000000 --threads 1

# --fetch-size 指定每次fetch的数据的大小
# --messages 总共要消费的消息个数

```



结果如下：

```shell
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec, rebalance.time.ms, fetch.time.ms, fetch.MB.sec, fetch.nMsg.sec
2019-08-03 19:19:24:566, 2019-08-03 19:19:40:258, 9.5367, 0.6077, 100000, 6372.6740, 3123, 12569, 0.7588, 7956.0824

开始测试时间，测试结束时间，最大吞吐率9.5367MB/s，平均每秒
```



## 4, 消费kafka的flume

* 我们这里直接从kafka消费，写入到hdfs中去
* ⚠️：为了避免产生小文件，需要设置 a1.sinks.k1.hdfs.minBlockReplicas = 1

```properties
## 组件
a1.sources = r1 r2
a1.channels = c1 c2
a1.sinks = k1 k2

## source1
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.zookeeperConnect = centos01:2181,centos02:2181,centos03:2181
a1.sources.r1.topic = topic_start
a1.sources.r1.groupId = flume
a1.sources.r1.batchSize = 5000
a1.sources.r1.batchDurationMillis = 2000

## source2
a1.sources.r2.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r2.zookeeperConnect = centos01:2181,centos02:2181,centos03:2181
a1.sources.r2.topic = topic_event
a1.sources.r2.groupId = flume
a1.sources.r2.batchSize = 5000
a1.sources.r2.batchDurationMillis = 2000


## channel1
a1.channels.c1.type = memory
a1.channels.c1.capacity = 100000
a1.channels.c1.transactionCapacity = 10000

## channel2
a1.channels.c2.type = memory
a1.channels.c2.capacity = 100000
a1.channels.c2.transactionCapacity = 10000

## sink1
a1.sinks.k1.type = hdfs
# a1.sinks.k1.hdfs.writeFormat = Text
a1.sinks.k1.hdfs.useLocalTimeStamp = true
a1.sinks.k1.hdfs.path = /flume/events/log/topic_start/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = logstart-
a1.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.roundValue = 10
a1.sinks.k1.hdfs.roundUnit = second

##sink2
a1.sinks.k2.type = hdfs
# a1.sinks.k2.hdfs.writeFormat = Text
a1.sinks.k2.hdfs.useLocalTimeStamp = true
a1.sinks.k2.hdfs.path = /flume/events/log/topic_event/%Y-%m-%d
a1.sinks.k2.hdfs.filePrefix = logevent-
a1.sinks.k2.hdfs.round = true
a1.sinks.k2.hdfs.roundValue = 10
a1.sinks.k2.hdfs.roundUnit = second

## 不要产生大量小文件
a1.sinks.k1.hdfs.minBlockReplicas = 1
a1.sinks.k1.hdfs.callTimeout = 100000
a1.sinks.k1.hdfs.rollInterval = 60
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0

a1.sinks.k2.hdfs.minBlockReplicas = 1
a1.sinks.k2.hdfs.callTimeout = 100000
a1.sinks.k2.hdfs.rollInterval = 60
a1.sinks.k2.hdfs.rollSize = 134217728
a1.sinks.k2.hdfs.rollCount = 0

## 控制输出文件是原生文件。
# a1.sinks.k1.hdfs.fileType = DataStream 
# a1.sinks.k2.hdfs.fileType = DataStream 
a1.sinks.k1.hdfs.fileType = CompressedStream 
a1.sinks.k2.hdfs.fileType = CompressedStream 

a1.sinks.k1.hdfs.codeC = lzop
a1.sinks.k2.hdfs.codeC = lzop

## 拼装
a1.sources.r1.channels = c1
a1.sinks.k1.channel= c1

a1.sources.r2.channels = c2
a1.sinks.k2.channel= c2
```



## 5, ods层：cdh上加载flume写入的数据

### 5.1, 创建数据库

```sql
create database gmall;
```



### 5.2,ods层创建表

* start表

```sql
drop table if exists gmall.ods_start_log;
CREATE EXTERNAL TABLE gmall.ods_start_log (`line` string)
PARTITIONED BY (`dt` string)
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_start_log';
```

* event表

```sql
drop table if exists gmall.ods_event_log;
CREATE EXTERNAL TABLE gmall.ods_event_log (`line` string)
PARTITIONED BY (`dt` string)
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_event_log';
```



### 5.3, ods层加载数据

* 加载start表

```sql
load data inpath '/flume/events/log/topic_start/2019-08-04' into table gmall.ods_start_log partition(dt='2019-08-04');
```

* 加载event表

```sql
load data inpath '/flume/events/log/topic_event/2019-08-04' into table gmall.ods_event_log partition(dt='2019-08-04');
```



### 5.4, ods层命令行自动导入

```shell
# hive -e是外部调用hive命令，可以通过这个命令在shell中执行load data命令即可
hive -e show databases;
```

```shell
#!/bin/bash
hive=/opt/cloudera/parcels/CDH-5.12.1-1.cdh5.12.1.p0.3/bin/hive

if [ -n "$1" ];
then theDate=$1
else theDate=`date -d "-1 day" +%F`
fi

echo $theDate

action="
load data inpath '/flume/events/log/topic_start/$theDate' into table gmall.ods_start_log partition(dt='$theDate');

load data inpath '/flume/events/log/topic_event/$theDate' into table gmall.ods_event_log partition(dt='$theDate');
"

echo $action

$hive -e "$action"
```



## 6, dwd层第一层: 创表导入等

### 6.0, 两个自定义hive函数：base_analizer和flat_analizer

* 两个函数：
* base_analizer是udf函数
* flat_analizer是udtf函数

#### 6.0.1, 首先加入依赖

```xml
<dependency>
  <groupId>org.apache.hive</groupId>
  <artifactId>hive-cli</artifactId>
  <version>1.1.0</version>
</dependency>
```

#### 6.0.2, udf函数代码编写

```java
package com.atguigu.udf;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.hive.ql.exec.UDF;
import org.json.JSONException;
import org.json.JSONObject;

/**
 * #author      : ivanl001
 * #creator     : 2019-08-05 10:04
 * #description :
 **/
public class EventBaseUDF extends UDF {

    //这个不是父类的方法，但是需要有这个方法，主要有这种形式即可
    public String evaluate(String line, String key) throws JSONException {

        // 0, 初始化返回结果串
        StringBuilder sb = new StringBuilder();

        // 1, 先对key进行切割，方便后面调用
        String[] jsonkeys = key.split(",");

        // 2, 处理获取的字符串
        if (line== null){
            return "";
        }
        String[] logContents = line.split("\\|");
        if (logContents.length != 2 || StringUtils.isBlank(logContents[1])){
            return "";
        }

        // 3, 创建json对象
        try {

            JSONObject jsonObject = new JSONObject(logContents[1]);
            //这里只获取cm字段内容分析，其他的字段作为后续处理
            JSONObject cm = jsonObject.getJSONObject("cm");
            for (int i = 0; i < jsonkeys.length; i++) {
                String jsonkey = jsonkeys[i];

                if(cm.has(jsonkey)){
                    sb.append(cm.getString(jsonkey)).append("\t");
                }else {
                    sb.append("\t");
                }
            }
            //把其他需要的字段拼接上
            sb.append(jsonObject.getString("et")).append("\t");
            sb.append(logContents[0]).append("\t");
        } catch (JSONException e) {
            e.printStackTrace();
        }
        return sb.toString();
    }


    public static void main(String[] args) throws JSONException {

        String line = "1541217850324|{\"cm\":{\"mid\":\"m7856\",\"uid\":\"u8739\",\"ln\":\"-74.8\",\"sv\":\"V2.2.2\",\"os\":\"8.1.3\",\"g\":\"P7XC9126@gmail.com\",\"nw\":\"3G\",\"l\":\"es\",\"vc\":\"6\",\"hw\":\"640*960\",\"ar\":\"MX\",\"t\":\"1541204134250\",\"la\":\"-31.7\",\"md\":\"huawei-17\",\"vn\":\"1.1.2\",\"sr\":\"O\",\"ba\":\"Huawei\"},\"ap\":\"weather\",\"et\":[{\"ett\":\"1541146624055\",\"en\":\"display\",\"kv\":{\"goodsid\":\"n4195\",\"copyright\":\"ESPN\",\"content_provider\":\"CNN\",\"extend2\":\"5\",\"action\":\"2\",\"extend1\":\"2\",\"place\":\"3\",\"showtype\":\"2\",\"category\":\"72\",\"newstype\":\"5\"}},{\"ett\":\"1541213331817\",\"en\":\"loading\",\"kv\":{\"extend2\":\"\",\"loading_time\":\"15\",\"action\":\"3\",\"extend1\":\"\",\"type1\":\"\",\"type\":\"3\",\"loading_way\":\"1\"}},{\"ett\":\"1541126195645\",\"en\":\"ad\",\"kv\":{\"entry\":\"3\",\"show_style\":\"0\",\"action\":\"2\",\"detail\":\"325\",\"source\":\"4\",\"behavior\":\"2\",\"content\":\"1\",\"newstype\":\"5\"}},{\"ett\":\"1541202678812\",\"en\":\"notification\",\"kv\":{\"ap_time\":\"1541184614380\",\"action\":\"3\",\"type\":\"4\",\"content\":\"\"}},{\"ett\":\"1541194686688\",\"en\":\"active_background\",\"kv\":{\"active_source\":\"3\"}}]}";
        String x = new EventBaseUDF().evaluate(line, "mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,nw,ln,la,t");
        System.out.println(x);
    }
}
```



#### 6.0..3, udtf函数代码编写

```java
package com.atguigu.udtf;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;
import org.json.JSONArray;
import org.json.JSONException;

import java.util.ArrayList;
import java.util.List;

/**
 * #author      : ivanl001
 * #creator     : 2019-08-05 11:21
 * #description :
 **/
public class EventExplodeUDTF extends GenericUDTF {

    @Override
    public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {

        List<String> fieldNames = new ArrayList<>();
        List<ObjectInspector> fieldTypes = new ArrayList<>();

        fieldNames.add("event_name");
        fieldTypes.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);

        fieldNames.add("event_json");
        fieldTypes.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);

        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames,fieldTypes);

    }

    @Override
    public void process(Object[] args) throws HiveException {

        // 1, 获取数据
        String input = args[0].toString();

        // 2, 基本的数据校验
        if (StringUtils.isBlank(input)){
            return;
        }

        try {
            JSONArray jsonArray = new JSONArray(input);
            for (int i = 0; i < jsonArray.length(); i++) {

                String[] results = new String[2];
                try {
                    // 获取事件名称
                    //这个是event_name
                    results[0]= jsonArray.getJSONObject(i).getString("en");
                    //这个是event_json
                    results[1] =jsonArray.getString(i);

                } catch (JSONException e) {
                    e.printStackTrace();
                    continue;
                }
                // 把结果写出去
                forward(results);
            }

        } catch (JSONException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void close() throws HiveException {

    }

    public static void main(String[] args) {
        //这个好想没法验证对吧
    }
}
```

#### 6.0..4, 打包放入hive的lib目录下

```shell
mv /root/06-dataware/03-hiveUDF-1.0-SNAPSHOT.jar /opt/cloudera/parcels/CDH/lib/hive/lib/ 
```

#### 6.0.5, 加入jar包, 并创建临时函数

```shell
# 进入hive环境
hive
add jar /opt/cloudera/parcels/CDH/lib/hive/lib/03-hiveUDF-1.0-SNAPSHOT.jar
create temporary function base_analizer as 'com.atguigu.udf.EventBaseUDF';
create temporary function flat_analizer as 'com.atguigu.udtf.EventExplodeUDTF';
```





### 6.1, dwd层创建表

* start表

```sql
drop table if exists gmall.dwd_start_log;
CREATE EXTERNAL TABLE gmall.dwd_start_log(
`mid_id` string,
`user_id` string, 
`version_code` string, 
`version_name` string, 
`lang` string, 
`source` string, 
`os` string, 
`area` string, 
`model` string,
`brand` string, 
`sdk_version` string, 
`gmail` string, 
`height_width` string,  
`app_time` string,
`network` string, 
`lng` string, 
`lat` string, 
`entry` string, 
`open_ad_type` string, 
`action` string, 
`loading_time` string, 
`detail` string, 
`extend1` string
)
PARTITIONED BY (dt string)
location '/warehouse/gmall/dwd/dwd_start_log';
```

* event表

```mysql
# 这里选择用parquet格式，是为了学习一下而已
drop table if exists gmall.dwd_base_event_log;
CREATE EXTERNAL TABLE gmall.dwd_base_event_log(
`mid_id` string,
`user_id` string, 
`version_code` string, 
`version_name` string, 
`lang` string, 
`source` string, 
`os` string, 
`area` string, 
`model` string,
`brand` string, 
`sdk_version` string, 
`gmail` string, 
`height_width` string, 
`app_time` string, 
`network` string, 
`lng` string, 
`lat` string, 
`event_name` string, 
`event_json` string, 
`server_time` string)
PARTITIONED BY (`dt` string)
stored as parquet
location '/warehouse/gmall/dwd/dwd_base_event_log';
```



### 6.2, dwd层从ods层获取并插入数据

* start表

```sql
SET hive.exec.dynamic.partition.mode=nonstrict;

insert overwrite table gmall.dwd_start_log
PARTITION (dt)
select 
    get_json_object(line,'$.mid') mid_id,
    get_json_object(line,'$.uid') user_id,
    get_json_object(line,'$.vc') version_code,
    get_json_object(line,'$.vn') version_name,
    get_json_object(line,'$.l') lang,
    get_json_object(line,'$.sr') source,
    get_json_object(line,'$.os') os,
    get_json_object(line,'$.ar') area,
    get_json_object(line,'$.md') model,
    get_json_object(line,'$.ba') brand,
    get_json_object(line,'$.sv') sdk_version,
    get_json_object(line,'$.g') gmail,
    get_json_object(line,'$.hw') height_width,
    get_json_object(line,'$.t') app_time,
    get_json_object(line,'$.nw') network,
    get_json_object(line,'$.ln') lng,
    get_json_object(line,'$.la') lat,
    get_json_object(line,'$.entry') entry,
    get_json_object(line,'$.open_ad_type') open_ad_type,
    get_json_object(line,'$.action') action,
    get_json_object(line,'$.loading_time') loading_time,
    get_json_object(line,'$.detail') detail,
    get_json_object(line,'$.extend1') extend1,
    dt 
from gmall.ods_start_log where dt = '2019-08-05'
```

* event表
* 该表处理的时候用到了udf和udtf函数，具体看下面的内容
* 用到的两个函数：base_analizer和flat_analizer, 参考上面6.0的内容

```mysql
set hive.exec.dynamic.partition.mode=nonstrict;

insert overwrite table gmall.dwd_base_event_log partition (dt)
select
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
event_name,
event_json,
server_time,
dt
from
(
select
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[0]   as mid_id,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[1]   as user_id,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[2]   as version_code,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[3]   as version_name,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[4]   as lang,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[5]   as source,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[6]   as os,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[7]   as area,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[8]   as model,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[9]   as brand,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[10]  as sdk_version,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[11]  as gmail,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[12]  as height_width,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[13]  as app_time,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[14]  as network,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[15]  as lng,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[16]  as lat,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[17]  as ops,
split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[18]  as server_time,
dt
from gmall.ods_event_log where dt='2019-08-05' and base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la')<>'' 
) sdk_log lateral view flat_analizer(ops) tmp_k as event_name, event_json;
```







## 7, dwd层第二层：event表基础明细表

* 其实dwd层还有好多张明细表， 这里只创建导入一张，作为案例，其他的不创建了哈

### 7.1, dwd_display_log: 明细表创建

```mysql
drop table if exists gmall.dwd_display_log;
CREATE EXTERNAL TABLE gmall.dwd_display_log(
`mid_id` string,
`user_id` string,
`version_code` string,
`version_name` string,
`lang` string,
`source` string,
`os` string,
`area` string,
`model` string,
`brand` string,
`sdk_version` string,
`gmail` string,
`height_width` string,
`app_time` string,
`network` string,
`lng` string,
`lat` string,
`action` string,
`goodsid` string,
`place` string,
`extend1` string,
`category` string,
`server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/gmall/dwd/dwd_display_log/';
```

### 7.1, dwd_display_log: 明细表数据导入

```mysql
set hive.exec.dynamic.partition.mode=nonstrict;

insert overwrite table gmall.dwd_display_log partition(dt)
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.action') action,
get_json_object(event_json,'$.kv.goodsid') goodsid,
get_json_object(event_json,'$.kv.place') place,
get_json_object(event_json,'$.kv.extend1') extend1,
get_json_object(event_json,'$.kv.category') category,
server_time,
dt
from gmall.dwd_base_event_log 
where dt='2019-08-05' and event_name='display';
```



## 8, dws层

### 日活表

### 8.1, dws_uv_detail_day

```mysql
drop table if exists gmall.dws_uv_detail_day;
create external table gmall.dws_uv_detail_day
(
    `mid_id` string COMMENT '设备唯一标识',
    `user_id` string COMMENT '用户标识', 
    `version_code` string COMMENT '程序版本号', 
    `version_name` string COMMENT '程序版本名', 
    `lang` string COMMENT '系统语言', 
    `source` string COMMENT '渠道号', 
    `os` string COMMENT '安卓系统版本', 
    `area` string COMMENT '区域', 
    `model` string COMMENT '手机型号', 
    `brand` string COMMENT '手机品牌', 
    `sdk_version` string COMMENT 'sdkVersion', 
    `gmail` string COMMENT 'gmail', 
    `height_width` string COMMENT '屏幕宽高',
    `app_time` string COMMENT '客户端日志产生时的时间',
    `network` string COMMENT '网络模式',
    `lng` string COMMENT '经度',
    `lat` string COMMENT '纬度'
)
partitioned by (dt string)
stored as parquet
location '/warehouse/gmall/dws/dws_uv_detail_day';
```



```mysql
insert overwrite table gmall.dws_uv_detail_day partition(dt)
select  
    mid_id,
    concat_ws('|', collect_set(user_id)) user_id,
    concat_ws('|', collect_set(version_code)) version_code,
    concat_ws('|', collect_set(version_name)) version_name,
    concat_ws('|', collect_set(lang))lang,
    concat_ws('|', collect_set(source)) source,
    concat_ws('|', collect_set(os)) os,
    concat_ws('|', collect_set(area)) area, 
    concat_ws('|', collect_set(model)) model,
    concat_ws('|', collect_set(brand)) brand,
    concat_ws('|', collect_set(sdk_version)) sdk_version,
    concat_ws('|', collect_set(gmail)) gmail,
    concat_ws('|', collect_set(height_width)) height_width,
    concat_ws('|', collect_set(app_time)) app_time,
    concat_ws('|', collect_set(network)) network,
    concat_ws('|', collect_set(lng)) lng,
    concat_ws('|', collect_set(lat)) lat,
    dt
from gmall.dwd_start_log
where dt='2019-08-05'
group by mid_id, dt;
```



### 周活表

### 8.2, dws_uv_detail_wk

```mysql
drop table if exists gmall.dws_uv_detail_wk;
create external table gmall.dws_uv_detail_wk( 
    `mid_id` string COMMENT '设备唯一标识',
    `user_id` string COMMENT '用户标识', 
    `version_code` string COMMENT '程序版本号', 
    `version_name` string COMMENT '程序版本名', 
    `lang` string COMMENT '系统语言', 
    `source` string COMMENT '渠道号', 
    `os` string COMMENT '安卓系统版本', 
    `area` string COMMENT '区域', 
    `model` string COMMENT '手机型号', 
    `brand` string COMMENT '手机品牌', 
    `sdk_version` string COMMENT 'sdkVersion', 
    `gmail` string COMMENT 'gmail', 
    `height_width` string COMMENT '屏幕宽高',
    `app_time` string COMMENT '客户端日志产生时的时间',
    `network` string COMMENT '网络模式',
    `lng` string COMMENT '经度',
    `lat` string COMMENT '纬度',
    `monday_date` string COMMENT '周一日期',
    `sunday_date` string COMMENT  '周日日期' 
) COMMENT '活跃用户按周明细'
PARTITIONED BY (`wk_dt` string)
stored as parquet
location '/warehouse/gmall/dws/dws_uv_detail_wk/';
```



```mysql
set hive.exec.dynamic.partition.mode=nonstrict;

insert overwrite table gmall.dws_uv_detail_wk partition(wk_dt)
select  
    mid_id,
    concat_ws('|', collect_set(user_id)) user_id,
    concat_ws('|', collect_set(version_code)) version_code,
    concat_ws('|', collect_set(version_name)) version_name,
    concat_ws('|', collect_set(lang)) lang,
    concat_ws('|', collect_set(source)) source,
    concat_ws('|', collect_set(os)) os,
    concat_ws('|', collect_set(area)) area, 
    concat_ws('|', collect_set(model)) model,
    concat_ws('|', collect_set(brand)) brand,
    concat_ws('|', collect_set(sdk_version)) sdk_version,
    concat_ws('|', collect_set(gmail)) gmail,
    concat_ws('|', collect_set(height_width)) height_width,
    concat_ws('|', collect_set(app_time)) app_time,
    concat_ws('|', collect_set(network)) network,
    concat_ws('|', collect_set(lng)) lng,
    concat_ws('|', collect_set(lat)) lat,
    date_add(next_day('2019-08-05','MO'),-7),
    date_add(next_day('2019-08-05','MO'),-1),
    concat(date_add( next_day('2019-08-05','MO'),-7), '_' , date_add(next_day('2019-08-05','MO'),-1) 
)
from gmall.dws_uv_detail_day 
where dt>=date_add(next_day('2019-08-05','MO'),-7) 
and dt<=date_add(next_day('2019-08-05','MO'),-1) 
group by mid_id;
```

### 月活表

### 8.3, dws_uv_detail_mn

```mysql
drop table if exists gmall.dws_uv_detail_mn;

create external table gmall.dws_uv_detail_mn( 
    `mid_id` string COMMENT '设备唯一标识',
    `user_id` string COMMENT '用户标识', 
    `version_code` string COMMENT '程序版本号', 
    `version_name` string COMMENT '程序版本名', 
    `lang` string COMMENT '系统语言', 
    `source` string COMMENT '渠道号', 
    `os` string COMMENT '安卓系统版本', 
    `area` string COMMENT '区域', 
    `model` string COMMENT '手机型号', 
    `brand` string COMMENT '手机品牌', 
    `sdk_version` string COMMENT 'sdkVersion', 
    `gmail` string COMMENT 'gmail', 
    `height_width` string COMMENT '屏幕宽高',
    `app_time` string COMMENT '客户端日志产生时的时间',
    `network` string COMMENT '网络模式',
    `lng` string COMMENT '经度',
    `lat` string COMMENT '纬度'
) COMMENT '活跃用户按月明细'
PARTITIONED BY (`mn` string)
stored as parquet
location '/warehouse/gmall/dws/dws_uv_detail_mn/';
```



```mysql
set hive.exec.dynamic.partition.mode=nonstrict;

insert overwrite table gmall.dws_uv_detail_mn partition(mn)
select  
    mid_id,
    concat_ws('|', collect_set(user_id)) user_id,
    concat_ws('|', collect_set(version_code)) version_code,
    concat_ws('|', collect_set(version_name)) version_name,
    concat_ws('|', collect_set(lang)) lang,
    concat_ws('|', collect_set(source)) source,
    concat_ws('|', collect_set(os)) os,
    concat_ws('|', collect_set(area)) area, 
    concat_ws('|', collect_set(model)) model,
    concat_ws('|', collect_set(brand)) brand,
    concat_ws('|', collect_set(sdk_version)) sdk_version,
    concat_ws('|', collect_set(gmail)) gmail,
    concat_ws('|', collect_set(height_width)) height_width,
    concat_ws('|', collect_set(app_time)) app_time,
    concat_ws('|', collect_set(network)) network,
    concat_ws('|', collect_set(lng)) lng,
    concat_ws('|', collect_set(lat)) lat,
    date_format('2019-08-05','yyyy-MM')
from gmall.dws_uv_detail_day
where date_format(dt,'yyyy-MM') = date_format('2019-08-05','yyyy-MM')
group by mid_id;
```

## 9, ads层

### 活跃用户表

### ads_uv_count

```mysql
drop table if exists gmall.ads_uv_count;
create external table gmall.ads_uv_count( 
    `dt` string COMMENT '统计日期',
    `day_count` bigint COMMENT '当日用户数量',
    `wk_count`  bigint COMMENT '当周用户数量',
    `mn_count`  bigint COMMENT '当月用户数量',
    `is_weekend` string COMMENT 'Y,N是否是周末,用于得到本周最终结果',
    `is_monthend` string COMMENT 'Y,N是否是月末,用于得到本月最终结果' 
) COMMENT '活跃设备数'
row format delimited fields terminated by '\t'
location '/warehouse/gmall/ads/ads_uv_count/';
```



```mysql
insert into table gmall.ads_uv_count 
select '2019-08-05' dt, daycount.ct, wkcount.ct, mncount.ct,
   if(date_add(next_day('2019-08-05','MO'),-1)='2019-08-05','Y','N') ,
   if(last_day('2019-08-05')='2019-08-05','Y','N') 
from 
(
   select '2019-08-05' dt, count(*) ct
   from gmall.dws_uv_detail_day
   where dt='2019-08-05'  
) as daycount 

join

( 
   select '2019-08-05' dt, count (*) ct
   from gmall.dws_uv_detail_wk
   where wk_dt=concat(date_add(next_day('2019-08-05','MO'),-7),'_' ,date_add(next_day('2019-08-05','MO'),-1) )
) as wkcount 

on daycount.dt=wkcount.dt

join 

( 
   select '2019-08-05' dt, count (*) ct
   from gmall.dws_uv_detail_mn
   where mn=date_format('2019-08-05','yyyy-MM')  
) as mncount 

on daycount.dt=mncount.dt;
```

### 新增用户表

### dws_new_mid_day

```mysql
drop table if exists gmall.dws_new_mid_day;
create external table gmall.dws_new_mid_day
(
    `mid_id` string COMMENT '设备唯一标识',
    `user_id` string COMMENT '用户标识', 
    `version_code` string COMMENT '程序版本号', 
    `version_name` string COMMENT '程序版本名', 
    `lang` string COMMENT '系统语言', 
    `source` string COMMENT '渠道号', 
    `os` string COMMENT '安卓系统版本', 
    `area` string COMMENT '区域', 
    `model` string COMMENT '手机型号', 
    `brand` string COMMENT '手机品牌', 
    `sdk_version` string COMMENT 'sdkVersion', 
    `gmail` string COMMENT 'gmail', 
    `height_width` string COMMENT '屏幕宽高',
    `app_time` string COMMENT '客户端日志产生时的时间',
    `network` string COMMENT '网络模式',
    `lng` string COMMENT '经度',
    `lat` string COMMENT '纬度'
)  COMMENT '每日新增设备信息'
PARTITIONED BY (`dt` string)
stored as parquet
location '/warehouse/gmall/dws/dws_new_mid_day/';
```



```mysql
set hive.exec.dynamic.partition.mode=nonstrict;

insert into table gmall.dws_new_mid_day partition (dt)

select a01.mid_id, a01.user_id, a01.version_code, a01.version_name, a01.lang, a01.source, a01.os, a01.area, a01.model, a01.brand, a01.sdk_version, a01.gmail, 
a01.height_width, a01.app_time, a01.network, a01.lng, a01.lat, a01.dt
from

(select mid_id, user_id, version_code, version_name, lang, source, os, area, model, brand, sdk_version, gmail, height_width, app_time, network, lng, lat, dt
from gmall.dws_uv_detail_day
where dt = '2019-08-05') as a01 

left join 

(select mid_id, user_id, version_code, version_name, lang, source, os, area, model, brand, sdk_version, gmail, height_width, app_time, network, lng, lat, dt
from gmall.dws_new_mid_day) as a02

on a01.mid_id = a02.mid_id and a02.mid_id is null
```



## 10, ads层即最后一层

* 在这一层完成之后通过sqoop把数据导出mysql即可
* 当然也可以像元聚公司一样，通过presto的java api，让后台人员直接自己拉取自己需要的数据
* 完成！