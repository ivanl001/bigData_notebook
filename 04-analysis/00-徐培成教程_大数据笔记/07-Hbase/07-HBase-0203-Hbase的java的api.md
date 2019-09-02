## 1, 准备工作



* java api连接hbase，之需要指定zk地址即可，因为hbase的基本信息都存在zk上

* 01， maven依赖
```xml
<!--具体版本视情况而定-->
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-server</artifactId>
    <version>1.2.3</version>
</dependency>
```

* 02，拷贝hbase中的hbase-site.xml配置文件到resource中
```xml
<configuration>
	<property>
	    <name>hbase.cluster.distributed</name>
	    <value>true</value>
	</property>

	<!--指定hbase数据在hdfs上的存放路径-->
	<!-- <property>
	    <name>hbase.rootdir</name>
	    <value>hdfs://centos01:8020/hbase</value>
	</property> -->

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

## 2, 插入api，包括单条插入和批量插入等
```java
package im.ivanl001.bigData.Hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.junit.Test;

import java.io.IOException;
import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.List;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-10 14:58
 * #description : HBase的基本测试
 **/
public class A01_Hbase_API_Test {


    //这个只是好奇，如果插入一亿条数据需要多久，不过没插入完成就关门了，所以就不再插入了
    @Test
    public void hbase_batch_listPut01() throws Exception{

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        Table table = connection.getTable(TableName.valueOf("ns1:t3"));

        DecimalFormat decimalFormat = new DecimalFormat();
        decimalFormat.applyPattern("000000000");

        long current01 = System.currentTimeMillis();

        List<Put> putList = new ArrayList<Put>();
        for (int i=0;i<100000000;i++) {

            String row = "r" + decimalFormat.format(i);
            String family = "f1";
            //qualaifier：中文是限定词的意思
            String qualifier = "name";
            String value = "ivanl" + decimalFormat.format(i);

            Put put = new Put(row.getBytes());
            put.addColumn(family.getBytes(), qualifier.getBytes(), value.getBytes());
            putList.add(put);

            if (i % 100000 == 0) {
                table.put(putList);
                putList.clear();
            }
        }

        //这里重新put一下防止因为最后的一个万一不整除出问题
        table.put(putList);

        long current02 = System.currentTimeMillis();
        System.out.println("costTime:" + (current02-current01));
        //这个只是好奇，如果插入一亿条数据需要多久，不过没插入完成就关门了，所以就不再插入了
    }


    @Test
    public void hbase_batch_listPut() throws Exception{

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        Table table = connection.getTable(TableName.valueOf("ns1:t2"));


        DecimalFormat decimalFormat = new DecimalFormat();
        decimalFormat.applyPattern("0000000");

        long current01 = System.currentTimeMillis();

        List<Put> putList = new ArrayList<Put>();
        for (int i=0;i<10;i++) {

            String row = "r" + decimalFormat.format(i);
            String family = "f1";
            //qualaifier：中文是限定词的意思
            String qualifier = "age";
            String value = decimalFormat.format(i);


            Put put = new Put(row.getBytes());
            put.addColumn(family.getBytes(), qualifier.getBytes(), value.getBytes());
            putList.add(put);

            if (i % 5000 == 0) {
                table.put(putList);
                putList.clear();
            }
        }

        //这里重新put一下防止因为最后的一个万一不整除出问题
        table.put(putList);

        long current02 = System.currentTimeMillis();
        System.out.println("costTime:" + (current02-current01));
        //costTime:31 717, 31秒，半分钟，哇，瞅瞅！！！！

    }


    //如果用单条put方式插入大批量数据，效率是不高的
    @Test
    public void hbase_batch_insert() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        Table table = connection.getTable(TableName.valueOf("ns1:t1"));

        DecimalFormat decimalFormat = new DecimalFormat();
        decimalFormat.applyPattern("0000000");

        long current01 = System.currentTimeMillis();

        for (int i=0;i<1000000;i++) {
            String row = "r" + decimalFormat.format(i);
            String family = "f1";
            //qualaifier：中文是限定词的意思
            String qualifier = "name";
            String value = "ivanl" + decimalFormat.format(i);

            Put put = new Put(row.getBytes());
            put.addColumn(family.getBytes(), qualifier.getBytes(), value.getBytes());

            table.put(put);
        }

        long current02 = System.currentTimeMillis();
        System.out.println("costTime:" + (current02-current01));
        //costTime:1449 474，大概是25分钟的样子，oh，myGod!
    }

    @Test
    public void decimalFormat_test() throws Exception {
        DecimalFormat decimalFormat = new DecimalFormat();
        //#对于整数部分没有影响, 对于小数部分：如果位数大于小数#个数，有几个#就保留几位，保留规则是大于5进一，否则舍去
        //decimalFormat.applyPattern("##.##")
        //如果#中间有其他符号，这是数字之间用来分割的符号，4,343,343,333.35
        //decimalFormat.applyPattern("###,###.##");
        //0：整数位不足个数补0，小数位不足个数也补0，0043333.30
        decimalFormat.applyPattern("0000000.00");
        System.out.println(decimalFormat.format(43333.3));
    }
    

    @Test
    public void hbase_get_test() throws Exception{

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        Table table = connection.getTable(TableName.valueOf("ns1:t1"));
        Get get = new Get("r2".getBytes());

        //get.addColumn("f1".getBytes(), "name".getBytes());

        Result result = table.get(get);
        byte[] value = result.getValue("f1".getBytes(), "name".getBytes());
        System.out.printf(new String(value));
    }


    //写入测试
    @Test
    public void hbase_put_test() throws IOException {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        Table table = connection.getTable(TableName.valueOf("ns1:t1"));
        Put put = new Put("r2".getBytes());

        put.addColumn("f1".getBytes(), "name".getBytes(), "Ivan Bool".getBytes());

        table.put(put);
    }
}
```

## 3, 管理表api，包括禁用，创建，删除， 以及扫描表等
```java
package im.ivanl001.bigData.Hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.junit.Test;

import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.NavigableMap;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-12 10:48
 * #description : hbaseApi
 **/
public class A02_Hbase_API02_Test {

    //获取所有列族的数据
    @Test
    public void hbase_scan03_test() throws Exception {
        
        Long start = System.currentTimeMillis();
        
        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t1");
        Table table = connection.getTable(tableName);

        //注意：这里一定要指定开始行或者结束行什么的， 要不然全局扫描很可能很悲剧
        //这里的开始行和结束行是前开后闭区间
        Scan scan = new Scan().withStartRow("r0000000".getBytes()).withStopRow("r0010000".getBytes());

        //也就是一次next请求会缓存多少数据，数据大一点，可以减少网络请求, 默认是不缓存的
        //scan.setCaching(1000);
        System.out.println(scan.getCaching());


        ResultScanner resultScanner = table.getScanner(scan);

        Iterator<Result> results = resultScanner.iterator();
        while (results.hasNext()) {
            Result result = results.next();

            /*byte[] value = result.getValue("f1".getBytes(), "name".getBytes());
            System.out.println(new String(value));*/
            //因为我们这里想要获取所有列族的数据，所以不能像上面那样指定具体的某一列族了
            NavigableMap<byte[], NavigableMap<byte[], NavigableMap<Long, byte[]>>> tMap = result.getMap();

            for (Map.Entry<byte[], NavigableMap<byte[], NavigableMap<Long, byte[]>>> entry : tMap.entrySet()) {

                byte[] familyKey = entry.getKey();
                NavigableMap<byte[], NavigableMap<Long, byte[]>> familyValue = entry.getValue();


                for (Map.Entry<byte[], NavigableMap<Long, byte[]>> entry1 : familyValue.entrySet()) {

                    byte[] columnKey = entry1.getKey();
                    NavigableMap<Long, byte[]> columnValue = entry1.getValue();

                    //对于一个具体的列，可能还有不同的版本
                    for (Map.Entry<Long, byte[]> entry2 : columnValue.entrySet()) {

                        Long versionKey = entry2.getKey();
                        byte[] versionValue = entry2.getValue();

                        System.out.println("列族："+ new String(familyKey) + ", 列：" + new String(columnKey) + ",版本号：" + versionKey + ",值是：" + new String(versionValue));

                    }
                }
            }
        }
        Long end = System.currentTimeMillis();

        System.out.println("完结撒花～～～ , 耗时:"  + (end-start));
        //完结撒花～～～ , 耗时:27085
        //完结撒花～～～ , 耗时:26752
        //差别也不是很大，不知道为啥教程里差别那么大
    }
    

    //这里要去获取所有的某一列族的数据, 主要看while里面的代码
    @Test
    public void hbase_scan02_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t1");

        Table table = connection.getTable(tableName);

        //注意：这里一定要指定开始行或者结束行什么的， 要不然全局扫描很可能很悲剧
        //这里的开始行和结束行是前开后闭区间
        Scan scan = new Scan().withStartRow("r0000000".getBytes()).withStopRow("r0000060".getBytes());
        ResultScanner resultScanner = table.getScanner(scan);

        Iterator<Result> results = resultScanner.iterator();
        while (results.hasNext()) {
            Result result = results.next();


            /*byte[] value = result.getValue("f1".getBytes(), "name".getBytes());
            System.out.println(new String(value));*/
            //因为我们这里想要获取某一列族的所有数据，所以不能像上面那样指定具体的某一列族和某一列了
            Map<byte[], byte[]> familyMap = result.getFamilyMap("f1".getBytes());

            for (Map.Entry<byte[], byte[]> entry : familyMap.entrySet()) {
                byte[] key = entry.getKey();
                byte[] value = entry.getValue();
                System.out.println(new String(key));
                System.out.println(new String(value));
            }
        }

        System.out.println("finished");
    }
    


    //扫描获取某一列族中某一列的值
    @Test
    public void hbase_scan01_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t1");

        Table table = connection.getTable(tableName);

        //注意：这里一定要指定开始行或者结束行什么的， 要不然全局扫描很可能很悲剧
        //这里的开始行和结束行是前开后闭区间
        Scan scan = new Scan().withStartRow("r0000000".getBytes()).withStopRow("r0000060".getBytes());
        ResultScanner resultScanner = table.getScanner(scan);

        Iterator<Result> results = resultScanner.iterator();
        while (results.hasNext()) {
            Result result = results.next();
            byte[] value = result.getValue("f1".getBytes(), "name".getBytes());
            System.out.println(new String(value));
        }
    }


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
    

    @Test
    public void hbase_dropTable_test() throws Exception{
        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        Admin admin = connection.getAdmin();
        admin.deleteTable(TableName.valueOf("ns2:t1"));

        List<TableDescriptor> tableDescriptors = admin.listTableDescriptors();
        for (TableDescriptor tableDescriptor : tableDescriptors) {
            System.out.println(tableDescriptor.getTableName());
        }

        System.out.println("over");

    }


    @Test
    public void hbase_disableTable_test() throws Exception {
        Configuration conf = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(conf);

        Admin admin = connection.getAdmin();

        admin.disableTable(TableName.valueOf("ns2:t1"));
        System.out.println("over");
    }



    @Test
    public void habse_createTable_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        Admin admin = connection.getAdmin();
        TableName tableName = TableName.valueOf("ns2:t1");

        //TableDescriptor tableDescriptor = TableDescriptorBuilder.newBuilder(tableName).build();

        HTableDescriptor tableDescriptor = new HTableDescriptor(tableName);
        HColumnDescriptor columnDescriptor = new HColumnDescriptor("f1");
        tableDescriptor.addFamily(columnDescriptor);

        //这个是保留删除的cell，但是这个boolean值就是不是有点多余
        //columnDescriptor.setKeepDeletedCells(KeepDeletedCells.TRUE);


        admin.createTable(tableDescriptor);

        List<TableDescriptor> tables = admin.listTableDescriptors();
        for (TableDescriptor table : tables) {
            System.out.println(table.getTableName());
        }

        System.out.println("finished");
    }

    
    @Test
    public void hbase_createNamespace_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        Admin admin = connection.getAdmin();
        NamespaceDescriptor namespaceDescriptor = NamespaceDescriptor.create("ns2").build();
        admin.createNamespace(namespaceDescriptor);

        NamespaceDescriptor[] descriptors = admin.listNamespaceDescriptors();
        for (NamespaceDescriptor descriptor : descriptors) {
            System.out.println(descriptor.getName());
        }
    }
}
```

## 4, 其他
```java
package im.ivanl001.bigData.Hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.junit.Test;

import java.util.Iterator;
import java.util.List;
import java.util.Map;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-12 16:15
 * #description : 版本相关的
 **/
public class A04_Hbase_API03_Test {


    //这里要去获取所有的某一列族的数据, 主要看while里面的代码
    @Test
    public void hbase_get_versions_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns2:t4");
        Table table = connection.getTable(tableName);

        Get get = new Get("r0001".getBytes());
        get.addColumn("f1".getBytes(), "name".getBytes());
        //我们系统设置的版本是3，所以这里没必要设置版本号大于3，因为即使是大于3，也不可能查出来大于3条的结果出来的
        get.readVersions(3);

        Result result = table.get(get);

        List<Cell> cells = result.getColumnCells("f1".getBytes(), "name".getBytes());
        for (Cell cell : cells) {

            Long stamp = cell.getTimestamp();
            cell.getValueLength();
            cell.getValueOffset();

            byte[] valueArr = cell.getValueArray();
            int start = cell.getValueOffset();
            int length = cell.getValueLength();

            byte[] value = new byte[length];
            System.arraycopy(valueArr, start, value, 0, length);

            System.out.println("stamp:" + stamp + ", value:" + new String(value));
        }

        //scan是不能设置版本的，这里不能用scan，只能用get
        //注意：这里一定要指定开始行或者结束行什么的， 要不然全局扫描很可能很悲剧
        //这里的开始行和结束行是前开后闭区间
        /*Scan scan = new Scan();
        ResultScanner resultScanner = table.getScanner(scan);
        Iterator<Result> results = resultScanner.iterator();
        while (results.hasNext()) {
            Result result = results.next();
            *//*byte[] value = result.getValue("f1".getBytes(), "name".getBytes());
            System.out.println(new String(value));*//*
            //因为我们这里想要获取某一列族的所有数据，所以不能像上面那样指定具体的某一列族和某一列了
            Map<byte[], byte[]> familyMap = result.getFamilyMap("f1".getBytes());

            for (Map.Entry<byte[], byte[]> entry : familyMap.entrySet()) {
                byte[] key = entry.getKey();
                byte[] value = entry.getValue();
                System.out.println(new String(key));
                System.out.println(new String(value));
            }
        }*/
        System.out.println("finished");
    }
}
```

