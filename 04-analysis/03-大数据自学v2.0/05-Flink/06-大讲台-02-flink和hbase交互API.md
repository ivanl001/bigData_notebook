> flink与hbase交互有两种api，一种是继承RichSourceFunction

| -    | -                                 |                  |
| ---- | --------------------------------- | ---------------- |
| 读取 | 方式1: 继承RichSourceFunction     | ✅                |
|      | 方式2: 继承TableInputFormat       | 报错，不知道为啥 |
| 写入 | 方式1: 继承RichSinkFunction       | 暂时未测试       |
|      | 方式2: 继承TableOutputFormat      | 暂时未测试       |
|      | 方式3: 直接使用HadoopOutputFormat | ✅                |

## 读取

### 方式1：继承RichSourceFunction

#### HbaseReader代码

```java
package im.ivanl002.batch.a00_flink_source;

import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.tuple.Tuple4;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.source.RichSourceFunction;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-26 11:05
 * #description :
 **/
public class HbaseReader extends RichSourceFunction<Tuple2<String, String>> {

    private Connection connection = null;
    private Table table = null;
    private Scan scan = null;

    private static final byte[] CF = "cf".getBytes();
    private static final byte[] ATTR = "attr".getBytes();


    @Override
    public void open(Configuration parameters) throws Exception {
        super.open(parameters);

        //1, 首先通过配置文件读取配置信息(注意：把hbase的配置文件hbase-site.xml拷贝到resources目录下)，创建连接池
        org.apache.hadoop.conf.Configuration configuration = HBaseConfiguration.create();

        //果然只需要配置zk就可以了
        configuration.set("hbase.zookeeper.quorum", "centos01:2181,centos02:2181,centos03:2181");
        configuration.set("hbase.zookeeper.property.clientPort", "2181");
        connection = ConnectionFactory.createConnection(configuration);

        //2, 通过连接池获取表
        // instantiate a Table instance
        table = connection.getTable(TableName.valueOf("test"));

        //3, 获取扫描器，并设置相关的属性，行，列，获取的范围(可选)
        scan = new Scan();
        scan.addFamily(CF);
        //scan.addColumn(CF, ATTR);

        //指定某一列
        //scan.setRowPrefixFilter(Bytes.toBytes("row1"));
        //指定开始和结束，前包后不包
        /*scan.setStartRow("".getBytes());
        scan.setStopRow("".getBytes());*/
    }

    @Override
    public void run(SourceContext<Tuple2<String, String>> ctx) throws Exception {

        ResultScanner rs = table.getScanner(scan);
        Iterator<Result> iterator = rs.iterator();

        Tuple2<String, String> tuple2 = new Tuple2<>();

        while (iterator.hasNext()) {

            Result result = iterator.next();
            String rowkey = Bytes.toString(result.getRow());
            StringBuffer valueBuffer = new StringBuffer();


            //一个cell是一个列族中的n多qulifier
            for (Cell cell : result.listCells()) {
                String attr = Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength());
                String value = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                valueBuffer.append(attr).append(",");
                valueBuffer.append(value).append(",");
            }

            tuple2.setFields(rowkey, valueBuffer.toString());
            ctx.collect(tuple2);
        }
    }

    @Override
    public void cancel() {
        try {
            if (table != null) {
                table.close();
            }
            if (connection != null) {
                connection.close();
            }
        } catch (IOException e) {

        }
    }
}
```

#### HbaseReader使用方式：

```java
package im.ivanl002.batch.a00_flink_source;

import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-26 11:49
 * #description : 注意：这种自定义数据源的方式只能是流处理方式哈
 **/
public class Flink_CustomSource_hbase {

    public static void main(String[] args) throws Exception {

        //1, 获取运行环境
        StreamExecutionEnvironment senv = StreamExecutionEnvironment.getExecutionEnvironment();

        //2, 添加数据源，具体可以看HbaseReader内部，里面把参数都配置好了
        DataStreamSource<Tuple2<String, String>> tuple2DataStreamSource = senv.addSource(new HbaseReader());
        tuple2DataStreamSource.print();

        //3, 执行
        senv.execute("hbaseReader");
    }
}
```



## 写入

### 方式3: 直接使用HadoopOutputFormat

```java
package im.ivanl002.batch.a99_flink_sink;

import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.hadoop.mapreduce.HadoopOutputFormat;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.tuple.Tuple4;
import org.apache.flink.configuration.ConfigConstants;
import org.apache.flink.configuration.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.Mutation;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.mapreduce.TableOutputFormat;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-26 09:18
 * #description :
 **/
public class Flink_sink_hbase {

    public static final byte[] CF = "cf".getBytes();
    //这个是列的名，如果有多个列，需要有多个名
    public static final byte[] NAME = "name".getBytes();
    public static final byte[] AGE = "age".getBytes();
    public static final byte[] ADDR = "addr".getBytes();

    public static void main(String[] args) throws Exception {

        //1, 获取运行环境
        ExecutionEnvironment benv = ExecutionEnvironment.getExecutionEnvironment();

        //2, 准备数据
        DataSet<Tuple4<String, String, Integer, String>> users = benv.fromElements(
                Tuple4.of("1000","zhansgan",30,"beijing"),
                Tuple4.of("1001","lisi",23,"guangdong"),
                Tuple4.of("1002","wangwu",45,"hubei"),
                Tuple4.of("1003","zhanliu",23,"beijing"),
                Tuple4.of("1004","lilei",56,"henan"),
                Tuple4.of("1005","maxiaoshuai",34,"xizang"),
                Tuple4.of("1006","liudehua",26,"fujian"),
                Tuple4.of("1007","jiangxiaohan",18,"hubei"),
                Tuple4.of("1008","qianjin",29,"shanxi"),
                Tuple4.of("1009","zhujie",37,"shandong"),
                Tuple4.of("1010","taobinzhe",19,"guangxi"),
Tuple4.of("1011","wuqixian",20,"hainan")
        );

        //3, 转换成hbase可插入格式
        MapOperator<Tuple4<String, String, Integer, String>, Tuple2<Text, Mutation>> mutationList = users.map(new RichMapFunction<Tuple4<String, String, Integer, String>, Tuple2<Text, Mutation>>() {

            Put put = null;
            Tuple2<Text, Mutation> result = null;

            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);
                result = new Tuple2<>();
            }

            @Override
            public Tuple2<Text, Mutation> map(Tuple4<String, String, Integer, String> value) throws Exception {
                String key = value.f0;
                String name = value.f1;
                Integer age = value.f2;
                String addr = value.f3;

                put = new Put(key.getBytes(ConfigConstants.DEFAULT_CHARSET));
                put.addColumn(CF, NAME, name.getBytes(ConfigConstants.DEFAULT_CHARSET));
                put.addColumn(CF, AGE, Bytes.toBytes(age));
                put.addColumn(CF, ADDR, addr.getBytes(ConfigConstants.DEFAULT_CHARSET));

                result.f0 = new Text(value.f0);
                result.f1 = put;
                return result;
            }
        });

        
        //4, 创建输出job并输出到hbase
        org.apache.hadoop.conf.Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum", "centos01,centos02,centos03");
        conf.set("hbase.zookeeper.property.clientPort", "2181");
        conf.set(TableOutputFormat.OUTPUT_TABLE, "test");
        /*conf.set("zookeeper.znode.parent", "/hbase-unsecure");*/
        conf.set("mapreduce.output.fileoutputformat.outputdir", "/tmp");
        Job job = Job.getInstance(conf);
        mutationList.output(new HadoopOutputFormat<Text, Mutation>(new TableOutputFormat<Text>(), job));

        //5, 开始执行
        benv.execute("TestWriteToHBase");
    }
}
```

