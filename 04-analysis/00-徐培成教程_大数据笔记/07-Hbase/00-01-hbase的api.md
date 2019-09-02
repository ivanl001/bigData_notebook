> java api连接hbase，之需要指定zk地址即可，因为hbase的基本信息都存在zk上



## 1, DQL

### 1.1, scan+Cell方式解析

```java
package im.ivanl001.bigData.im.version2.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.Iterator;
import java.util.Map;
import java.util.NavigableMap;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-25 15:09
 * #description :
 **/
public class a01_Hbase_scan_Cell_test01 {

    public static final byte[] CF = "cf".getBytes();
    public static final byte[] ATTR = "attr".getBytes();


    public static void main(String[] args) throws IOException {

        //1, 首先通过配置文件读取配置信息(注意：把hbase的配置文件hbase-site.xml拷贝到resources目录下)，创建连接池
        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        //2, 通过连接池获取表
        // instantiate a Table instance
        Table table = connection.getTable(TableName.valueOf("test"));

        //3, 获取扫描器，并设置相关的属性，行，列，获取的范围(可选)
        Scan scan = new Scan();
        scan.addFamily(CF);
        //scan.addColumn(CF, ATTR);

        //指定某一列
        //scan.setRowPrefixFilter(Bytes.toBytes("row1"));
        //指定开始和结束，前包后不包
//        scan.setStartRow("row1".getBytes()).setStopRow("row43".getBytes());


        //4, 进行扫描，获取结果
        ResultScanner rs = table.getScanner(scan);
        Iterator<Result> iterator = rs.iterator();

        //5, 解析结果
        while (iterator.hasNext()) {

            //这是一行
            Result result = iterator.next();

            String rowkey = Bytes.toString(result.getRow());
            StringBuffer valueBuffer = new StringBuffer();

            for (Cell cell : result.listCells()) {

                String attr = Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength());
                valueBuffer.append(attr).append(",");

                String value = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                valueBuffer.append(value).append(",");
            }
//            String valueString = valueBuffer.replace(valueBuffer.length() - 1, valueBuffer.length(), "").toString();


            System.out.println("rowkey:-----" + rowkey + ", value:" + valueBuffer.toString());
        }
    }
}
```

### 1.2, scan+getMap方式解析

```java
package im.ivanl001.bigData.im.version2.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.Map;
import java.util.NavigableMap;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-25 15:09
 * #description :
 **/
public class a01_Hbase_scan_test {

    public static final byte[] CF = "cf".getBytes();
    public static final byte[] ATTR = "attr".getBytes();


    public static void main(String[] args) throws IOException {

        //1, 首先通过配置文件读取配置信息(注意：把hbase的配置文件hbase-site.xml拷贝到resources目录下)，创建连接池
        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        //2, 通过连接池获取表
        // instantiate a Table instance
        Table table = connection.getTable(TableName.valueOf("test"));

        //3, 获取扫描器，并设置相关的属性，行，列，获取的范围(可选)
        Scan scan = new Scan();
//        scan.addColumn(CF, ATTR);

        //指定某一列
        //scan.setRowPrefixFilter(Bytes.toBytes("row1"));
        //指定开始和结束，前包后不包
        scan.setStartRow("row1".getBytes()).setStopRow("row43".getBytes());


        //4, 进行扫描，获取结果
        ResultScanner rs = table.getScanner(scan);

        try {
            for (Result r = rs.next(); r != null; r = rs.next()) {

                // -----------------5.0, 解析结果，如果只想取出指定列的内容，可以直接下面这样即可-----------------
                //System.out.println(r);
                /*byte[] value = r.getValue(CF, "attr1".getBytes());
                System.out.println(new String(value));*/


                //-----------------如果想要把所有的列都取出来的话，那是需要用下面这种遍历的方式的哈-----------------
                //5.1, 因为我们这里想要获取所有列族的数据，所以不能像上面那样指定具体的某一列族了
                NavigableMap<byte[], NavigableMap<byte[], NavigableMap<Long, byte[]>>> tMap = r.getMap();

                //先便利看有多少个列族
                for (Map.Entry<byte[], NavigableMap<byte[], NavigableMap<Long, byte[]>>> entry : tMap.entrySet()) {

                    //取出列族key名
                    byte[] familyKey = entry.getKey();
                    //取出列族value
                    NavigableMap<byte[], NavigableMap<Long, byte[]>> familyValue = entry.getValue();

                    //一个列族可能有多个列，便利列
                    for (Map.Entry<byte[], NavigableMap<Long, byte[]>> entry1 : familyValue.entrySet()) {

                        //取出列key名
                        byte[] columnKey = entry1.getKey();
                        //取出列value值
                        NavigableMap<Long, byte[]> columnValue = entry1.getValue();


                        //对于一个具体的列，可能还有不同的版本
                        for (Map.Entry<Long, byte[]> entry2 : columnValue.entrySet()) {

                            Long versionKey = entry2.getKey();
                            byte[] versionValue = entry2.getValue();

                            System.out.println("列族：" + new String(familyKey) + ", 列：" + new String(columnKey) + ",版本号：" + versionKey + ",值是：" + new String(versionValue));
                        }
                    }
                }
            }
        } finally {
            rs.close();  // always close the ResultScanner!
        }
    }
}
```



### 1.3, get

```java
package im.ivanl001.bigData.im.version2.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-25 16:27
 * #description :
 **/
public class a02_Hbase_Get {


    public static final byte[] CF = "cf".getBytes();
    public static final byte[] ATTR = "attr1".getBytes();

    public static void main(String[] args) throws IOException {

        //1, 首先通过配置文件读取配置信息(注意：把hbase的配置文件hbase-site.xml拷贝到resources目录下)，创建连接池
        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        //2, 通过连接池获取表
        // instantiate a Table instance
        Table table = connection.getTable(TableName.valueOf("test"));

        //3, 设置取出需要到字段值value
        //row1是rowkey，判断是哪一行，cf是列族，判断是那一列族，attr是列名，判断是那一列，还有一个时间版本概念，默认是取最新
        Get get = new Get(Bytes.toBytes("row1"));
        //get.getMaxVersions();//这个是取最新值
        //get.setTimeRange();这个是取指定时间戳之间的数据

        //4, 取出结果
        Result r = table.get(get);
        byte[] b = r.getValue(CF, ATTR);  // returns current version of value
        System.out.println(new String(b));
    }
}
```

### 1.4, get+Versions

```java
package im.ivanl001.bigData.im.version2.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.KeyValue;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.List;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-25 16:41
 * #description :
 **/
public class a03_Hbase_Get_versions {

    public static final byte[] CF = "cf".getBytes();
    public static final byte[] ATTR = "attr".getBytes();

    public static void main(String[] args) throws IOException {

        //1, 首先通过配置文件读取配置信息(注意：把hbase的配置文件hbase-site.xml拷贝到resources目录下)，创建连接池
        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        //2, 通过连接池获取表
        // instantiate a Table instance
        Table table = connection.getTable(TableName.valueOf("test"));

        //3, 设置取出需要到字段值value
        //row1是rowkey，判断是哪一行，cf是列族，判断是那一列族，attr是列名，判断是那一列，还有一个时间版本概念，默认是取最新
        Get get = new Get(Bytes.toBytes("row1"));
        //get.getMaxVersions();//这个是取最新值
        //get.setTimeRange();这个是取指定时间戳之间的数据

        //4, 设置最大的版本号
        get.setMaxVersions(3);

        //5, 取出结果
        Result r = table.get(get);
        /*byte[] b = r.getValue(CF, ATTR);  // returns current version of value
        System.out.println("最新版本是：" + new String(b));*/


        //-----暂时不太会取出来，先放着哈-----
        List<KeyValue> kv = r.getColumn(CF, ATTR);  // returns all versions of this column
        for (KeyValue keyValue : kv) {
            System.out.println(new String(keyValue.getKey()) + ":" + new String(keyValue.getValueArray()));
        }

        List<Cell> cells = r.getColumnCells(CF, ATTR);  // returns all versions of this column
        for (Cell cell : cells) {
            System.out.println("最新的三个版本，版本号：" );
        }
    }
}
```




## 2, DDL

### 2.1, admin权限创建命名空间

```java
@Test
public void hbase_createNamespace_test() throws Exception {

  //1, 创建配置文件，并用配置创建连接
  Configuration configuration = HBaseConfiguration.create();
  Connection connection = ConnectionFactory.createConnection(configuration);

  //2, 根据连接获取管理员权限，创建命名空间
  Admin admin = connection.getAdmin();
  NamespaceDescriptor namespaceDescriptor = NamespaceDescriptor.create("ns2").build();
  admin.createNamespace(namespaceDescriptor);

  //3, 显示所有的命名空间
  NamespaceDescriptor[] descriptors = admin.listNamespaceDescriptors();
  for (NamespaceDescriptor descriptor : descriptors) {
    System.out.println(descriptor.getName());
  }
}
```



### 2.2, admin权限创建表

```java
@Test
public void habse_createTable_test() throws Exception {

  //1, 创建配置文件，并用配置创建连接
  Configuration configuration = HBaseConfiguration.create();
  Connection connection = ConnectionFactory.createConnection(configuration);

  //2, 根据连接获取管理员权限，创建表
  Admin admin = connection.getAdmin();
  TableName tableName = TableName.valueOf("ns2:t1");
  //TableDescriptor tableDescriptor = TableDescriptorBuilder.newBuilder(tableName).build();
  HTableDescriptor tableDescriptor = new HTableDescriptor(tableName);
  HColumnDescriptor columnDescriptor = new HColumnDescriptor("f1");
  tableDescriptor.addFamily(columnDescriptor);

  //这个是保留删除的cell，但是这个boolean值就是不是有点多余
  //columnDescriptor.setKeepDeletedCells(KeepDeletedCells.TRUE);

  admin.createTable(tableDescriptor);

  //3, 显示所有命名空间下的表
  //client的2.1版本有，但是1.2上没有
  /*List<TableDescriptor> tables = admin.listTableDescriptors();
        for (TableDescriptor table : tables) {
            System.out.println(table.getTableName());
        }*/
  HTableDescriptor[] tableDescriptors = admin.listTableDescriptorsByNamespace("ns2");
  for (HTableDescriptor tableDescriptor1 : tableDescriptors) {
    System.out.println(tableDescriptor1.getTableName());
  }

  System.out.println("finished");
}
```





### 2.3, 禁用表

```java
//禁用表
@Test
public void hbase_disableTable_test() throws Exception {

  Configuration conf = HBaseConfiguration.create();
  Connection connection = ConnectionFactory.createConnection(conf);

  Admin admin = connection.getAdmin();
  admin.disableTable(TableName.valueOf("ns2:t1"));
  System.out.println("over");
}
```



### 2.4, 删除表

```java
//删除表
@Test
public void hbase_dropTable_test() throws Exception{
  Configuration configuration = HBaseConfiguration.create();
  Connection connection = ConnectionFactory.createConnection(configuration);

  Admin admin = connection.getAdmin();
  admin.deleteTable(TableName.valueOf("ns2:t1"));

  //client的2.1版本有，但是1.2上没有
  /*List<TableDescriptor> tableDescriptors = admin.listTableDescriptors();
        for (TableDescriptor tableDescriptor : tableDescriptors) {
            System.out.println(tableDescriptor.getTableName());
        }*/
  HTableDescriptor[] tableDescriptors = admin.listTableDescriptorsByNamespace("ns2");
  for (HTableDescriptor tableDescriptor1 : tableDescriptors) {
    System.out.println(tableDescriptor1.getTableName());
  }

  System.out.println("over");
}
```

## 3, DML

### 3.1, put

* 这个里面根本不需要调用flush方法

```java
package im.ivanl001.bigData.im.version2.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-25 17:01
 * #description : 不需要调用flush方法
 **/
public class a04_Hbase_Put {

    public static final byte[] CF = "cf".getBytes();
    public static final byte[] ATTR = "attr".getBytes();

    public static void main(String[] args) throws IOException {

        //1, 首先通过配置文件读取配置信息(注意：把hbase的配置文件hbase-site.xml拷贝到resources目录下)，创建连接池
        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        //2, 通过连接池获取表
        // instantiate a Table instance
        Table table = connection.getTable(TableName.valueOf("test"));


        BufferedMutator btable = connection.getBufferedMutator(TableName.valueOf("test"));

        //3, 准备行key和数据
        String row = "row5";
        String data = "ivanl001 is the king of world!";

        //4, 写入到里面
        Put put = new Put(Bytes.toBytes(row));
        put.addColumn(CF, ATTR, Bytes.toBytes(data));
        table.put(put);


        //精准到时间戳，不过一般也没必要
        /*Put put = new Put( Bytes.toBytes(row));
        long explicitTimeInMs = 555;  // just an example
        put.add(CF, ATTR, explicitTimeInMs, Bytes.toBytes(data));
        table.put(put);*/
        /*
        Caution: the version timestamp is used internally by HBase for things like time-to-live calculations. It’s usually best to avoid setting this timestamp yourself. Prefer using a separate timestamp attribute of the row, or have the timestamp as a part of the row key, or both.
         */
    }
}
```



### 3.2, batch put(put方式)

```java
package im.ivanl001.bigData.im.version2.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-25 19:29
 * #description : 使用put做批处理操作,可以参考im.ivanl001.bigData.Hbase.A01_Hbase_API_Test#hbase_batch_listPut()做更进一步的更改
 **/
public class a05_Hbase_Put_batch {

    public static final byte[] CF = "cf".getBytes();
    public static final byte[] ATTR = "attr".getBytes();

    public static void main(String[] args) throws IOException {

        System.out.println("批量写入开始");

        //1, 首先通过配置文件读取配置信息(注意：把hbase的配置文件hbase-site.xml拷贝到resources目录下)，创建连接池
        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        //2, 通过连接池获取表
        // instantiate a Table instance
        Table table = connection.getTable(TableName.valueOf("test"));

        //3, 准备行key和数据
        List<Put> puts = new ArrayList<Put>();
        Put put = null;

        String row01 = "row6";
        String data01 = "ivanl006 is the king of world!";
        put = new Put(Bytes.toBytes(row01));
        put.addColumn(CF, ATTR, Bytes.toBytes(data01));
        puts.add(put);

        String row02 = "row7";
        String data02 = "ivanl007 is the king of world!";
        put = new Put(Bytes.toBytes(row02));
        put.addColumn(CF, ATTR, Bytes.toBytes(data02));
        puts.add(put);

        //4, 写入到里面
        table.put(puts);

        System.out.println("批量写入完成");
    }
}
```



### 3.3, 批量 BufferedMutator

```java
package im.ivanl001.bigData.im.version2.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-25 19:27
 * #description : 使用Mutator做批处理操作，这个是异步批处理的一个类
 *
 * Used to communicate with a single HBase table similar to Table but meant for batched, asynchronous puts.
 *
 **/
public class a06_Hbase_Mutator_batch_asynchronous {

    public static final byte[] CF = "cf".getBytes();
    public static final byte[] ATTR = "attr".getBytes();

    public static void main(String[] args) throws IOException {

        System.out.println("异步批量写入开始");

        //1, 首先通过配置文件读取配置信息(注意：把hbase的配置文件hbase-site.xml拷贝到resources目录下)，创建连接池
        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        //2, 通过连接池获取表
        // instantiate a Table instance,
        BufferedMutator btable = connection.getBufferedMutator(TableName.valueOf("test"));

        //3, 准备数据
        List<Mutation> mutationList = new ArrayList<Mutation>();

        Put put = null;
        String row01 = "row8";
        String data01 = "ivanl008 is the king of world!";
        put = new Put(Bytes.toBytes(row01));
        put.addColumn(CF, ATTR, Bytes.toBytes(data01));
        mutationList.add(put);


        String row02 = "row9";
        String data02 = "ivanl009 is the king of world!";
        put = new Put(Bytes.toBytes(row02));
        put.addColumn(CF, ATTR, Bytes.toBytes(data02));
        mutationList.add(put);

        //4, 批量写入数据
        btable.mutate(mutationList);

        if (btable != null) {
            btable.close();
        }

        System.out.println("异步批量写入结束");
    }
}
```







### 3.4, 删除数据

```java
//删除数据
//删除一条记录是不需要管理员权限的， 跟之前类似， 创建一个delete即可
@Test
public void hbase_deleteRow_test() throws Exception {

  Configuration configuration = HBaseConfiguration.create();
  Connection connection = ConnectionFactory.createConnection(configuration);

  Admin admin = connection.getAdmin();
  TableName tableName = TableName.valueOf("ns2:t1");
  Table table = connection.getTable(tableName);

  //先添加数据
  /*Put put = new Put("r01".getBytes());
        put.addColumn("f1".getBytes(), "name".getBytes(), "Ivan Bool".getBytes());
        table.put(put);*/


  //再删除数据
  Delete delete = new Delete("r01".getBytes());
  //这里不添加列的话默认删除当前一行， 如果添加，仅仅删除当前限制的列
  delete.addColumn("f1".getBytes(), "name".getBytes());
  table.delete(delete);

  //获取看一下结果是什么样子的
  /*Get get = new Get("r01".getBytes());
        Result result = table.get(get);
        System.out.println(result.value());*/
  //null，说明已经删除成功

}
```




