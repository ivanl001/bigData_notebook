*直接看代码中注释*

```java
package im.ivanl001.bigData.Hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.CompareOperator;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.filter.*;
import org.junit.Test;

import java.io.File;
import java.util.*;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-12 19:42
 * #description : 过滤器的使用
 **/
public class A05_Filter {


    //########------建议先看一下最下面那个， 那个比较全一些------#############

    //这里因为之前想同时设置两个filter的时候，后者会覆盖前者，所以不能成功，这里用FilterList可以实现，如下
    @Test
    public void hbase_familyFilter00000_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t2");
        Table table = connection.getTable(tableName);

        //终于明白了这里设置一个scan的作用是什么了，可以设置过滤器呀
        Scan scan = new Scan();
        //注意：如果添加列族，那么只会扫描指定列族的内容，如果没有添加，会扫描所有列族
        //scan.addColumn("f1".getBytes(), "name".getBytes());
        //scan.addColumn("f2".getBytes(), "age".getBytes());

        //
        Filter filter = new FamilyFilter(CompareOperator.EQUAL, new BinaryComparator("f2".getBytes()));
        Filter filter01 = new RowFilter(CompareOperator.LESS_OR_EQUAL, new BinaryComparator("r0001000".getBytes()));
        //如果连续添加过滤器，后者会覆盖前者！！！！！！！
        /*scan.setFilter(filter01);
        scan.setFilter(filter);*/


        List<Filter> filters = new ArrayList<Filter>();
        filters.add(filter);
        filters.add(filter01);


        //FilterList构造器需要一个Filter的list
        FilterList filterList = new FilterList(filters);


        scan.setFilter(filterList);


        ResultScanner resultScanner = table.getScanner(scan);
        Iterator<Result> results = resultScanner.iterator();

        while (results.hasNext()) {

            Result result = results.next();
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

        resultScanner.close();
    }


    //这个是个静态方法，懒得重复拷贝，所以直接抽取出来放在这里方便调用
    public static void func(Filter filter) throws Exception {
        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t4");
        Table table = connection.getTable(tableName);

        //终于明白了这里设置一个scan的作用是什么了，可以设置过滤器呀
        Scan scan = new Scan();
        scan.setFilter(filter);

        ResultScanner resultScanner = table.getScanner(scan);
        Iterator<Result> results = resultScanner.iterator();

        while (results.hasNext()) {

            Result result = results.next();
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
        resultScanner.close();
    }



    //实现like类似的功能
    @Test
    public void hbase_like_similar_test() throws Exception {

        //这个可以r002%这样的效果
        Filter filter = new ValueFilter(CompareOperator.EQUAL, new RegexStringComparator("^r002"));
        func(filter);

    }





    //大多类似，剩下的挑选进行代码演示
    @Test
    public void hbase_ColumnCountGetFilter_test() throws Exception {
        //取前2列
        Filter filter = new ColumnCountGetFilter(2);
        func(filter);
    }





    // 只有键值，没有值的
    @Test
    public void hbase_KeyOnlyFilter_test() throws Exception {
        //列族：f1, 列：name,版本号：1542074141465,值是：
        Filter filter = new KeyOnlyFilter();
        func(filter);

    }



    @Test
    public void hbase_PageFilter_test() throws Exception {

        //就取前1条， 这个是按区来限定的， 如果有三个区， 就是每个区上限取两个了
        Filter filter = new PageFilter(2);
        func(filter);
        //在这个基础上也可以使用scan的startwithrow还是叫什么的方法， 来限定开始行和结束行

    }







    @Test
    public void hbase_PrefixFilter_test() throws Exception {

        //r做前缀的， r001, 我这张表中添加了类似r001， r002， s001之类的rowkey
        Filter filter = new PrefixFilter("r".getBytes());//
        func(filter);

    }




    @Test
    public void hbase_singleComlumnValueExcludeFilter_test() throws Exception {

        //这里设定如果f1列族的age小于3，那么这一行就会被过滤掉, 该行作为过滤条件，也是不显示，也就是会被直接过滤掉
        Filter filter = new SingleColumnValueExcludeFilter("f1".getBytes(), "age".getBytes(), CompareOperator.GREATER, "30".getBytes());
        func(filter);


    }





    //单行值过滤，如果这个值不满足，整行都会被过滤掉，注意：过滤条件这个列也会被过滤掉，也就是不显示的哈
    @Test
    public void habse_singleColumnValueFilter_test() throws Exception {

        //这里设定如果f1列族的age小于3，那么这一行就会被过滤掉, 第二行全部显示， 可以参考对比下 上面的SingleColumnValueExcludeFilter
        Filter filter = new SingleColumnValueFilter("f1".getBytes(), "age".getBytes(), CompareOperator.GREATER, "30".getBytes());
        func(filter);
    }




    //这个居然每讲完😢😢😢
    @Test
    public void hbase_dependentColumnFilter_test() throws Exception {

        //这里好像也不对
        //不知道这个是干嘛用的， 怎么设置都没有东西，算了，估计也是废的
        Filter filter = new DependentColumnFilter("f1".getBytes(), "age".getBytes(), false, CompareOperator.LESS_OR_EQUAL, new BinaryComparator("10".getBytes()));
//        scan.setFilter(filter);
        func(filter);
    }






    //对get设置qualifierFilter过滤器
    @Test
    public void hbase_valueFileter01_test() throws Exception {


        //
        //Filter filter01 = new RowFilter(CompareOperator.LESS_OR_EQUAL, new BinaryComparator("r0001000".getBytes()));
        Filter filter = new ValueFilter(CompareOperator.LESS_OR_EQUAL, new BinaryComparator("ivanl0000100".getBytes()));



        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t2");
        Table table = connection.getTable(tableName);



        Get get = new Get("r0000001".getBytes());
        get.setFilter(filter);


        Result result = table.get(get);

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






    //对scan设置valueFileter过滤器
    @Test
    public void hbase_valueFileter_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t2");
        Table table = connection.getTable(tableName);

        //终于明白了这里设置一个scan的作用是什么了，可以设置过滤器呀
        Scan scan = new Scan();

        //
        Filter filter = new ValueFilter(CompareOperator.LESS_OR_EQUAL, new BinaryComparator("ivanl0000010".getBytes()));
        scan.setFilter(filter);


        ResultScanner resultScanner = table.getScanner(scan);
        Iterator<Result> results = resultScanner.iterator();

        while (results.hasNext()) {

            Result result = results.next();
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

        resultScanner.close();
    }






    //对get设置qualifierFilter过滤器
    @Test
    public void hbase_qualifierFilter01_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t2");
        Table table = connection.getTable(tableName);

        //
        //Filter filter01 = new RowFilter(CompareOperator.LESS_OR_EQUAL, new BinaryComparator("r0001000".getBytes()));
        Filter filter = new QualifierFilter(CompareOperator.EQUAL, new BinaryComparator("age".getBytes()));


        Get get = new Get("r0000001".getBytes());
        get.setFilter(filter);


        Result result = table.get(get);

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








    //对scan设置qualifierFilter过滤器
    @Test
    public void hbase_qualifierFilter_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t2");
        Table table = connection.getTable(tableName);

        //终于明白了这里设置一个scan的作用是什么了，可以设置过滤器呀
        Scan scan = new Scan();
        //注意：如果添加列族，那么只会扫描指定列族的内容，如果没有添加，会扫描所有列族
        //scan.addColumn("f1".getBytes(), "name".getBytes());
        //scan.addColumn("f2".getBytes(), "age".getBytes());

        //
        //Filter filter01 = new RowFilter(CompareOperator.LESS_OR_EQUAL, new BinaryComparator("r0001000".getBytes()));
        Filter filter = new QualifierFilter(CompareOperator.EQUAL, new BinaryComparator("age".getBytes()));

        //如果连续添加过滤器，后者会覆盖前者！！！！！！！
        //scan.setFilter(filter01);
        scan.setFilter(filter);


        ResultScanner resultScanner = table.getScanner(scan);
        Iterator<Result> results = resultScanner.iterator();

        while (results.hasNext()) {

            Result result = results.next();
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

        resultScanner.close();
    }





    //对scan设置familyFilter过滤器
    @Test
    public void hbase_familyFilter01_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t2");
        Table table = connection.getTable(tableName);

        Get get = new Get("r0000001".getBytes());


        Filter filter = new FamilyFilter(CompareOperator.EQUAL, new BinaryComparator("f2".getBytes()));
        //Filter filter01 = new RowFilter(CompareOperator.LESS_OR_EQUAL, new BinaryComparator("r0001000".getBytes()));
        //这个就是在get上添加过滤器
        get.setFilter(filter);


        Result result = table.get(get);
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





    //对scan设置familyFilter过滤器
    @Test
    public void hbase_familyFilter_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t2");
        Table table = connection.getTable(tableName);

        //终于明白了这里设置一个scan的作用是什么了，可以设置过滤器呀
        Scan scan = new Scan();
        //注意：如果添加列族，那么只会扫描指定列族的内容，如果没有添加，会扫描所有列族
        //scan.addColumn("f1".getBytes(), "name".getBytes());
        //scan.addColumn("f2".getBytes(), "age".getBytes());

        //
        Filter filter = new FamilyFilter(CompareOperator.EQUAL, new BinaryComparator("f2".getBytes()));
        Filter filter01 = new RowFilter(CompareOperator.LESS_OR_EQUAL, new BinaryComparator("r0001000".getBytes()));
        //如果连续添加过滤器，后者会覆盖前者！！！！！！！
        scan.setFilter(filter01);
        scan.setFilter(filter);



        ResultScanner resultScanner = table.getScanner(scan);
        Iterator<Result> results = resultScanner.iterator();

        while (results.hasNext()) {

            Result result = results.next();
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

        resultScanner.close();


    }




    //对scan设置rowFilter过滤器
    @Test
    public void hbase_rowFilter_test() throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(configuration);

        TableName tableName = TableName.valueOf("ns1:t1");
        Table table = connection.getTable(tableName);

        //终于明白了这里设置一个scan的作用是什么了，可以设置过滤器呀
        Scan scan = new Scan();
        scan.addColumn("f1".getBytes(), "name".getBytes());

        //第一种，二进制对比器
        /*Filter filter = new RowFilter(CompareOperator.LESS_OR_EQUAL, new BinaryComparator("r0001000".getBytes()));
        scan.setFilter(filter);*/

        //第二种，正则对比器, 正则大多也不记得， 先放着吧
        /*Filter filter2 = new RowFilter(CompareOperator.EQUAL, new RegexStringComparator(""));
        scan.setFilter(filter2);*/

        //第三种，截串对比器
        Filter filter3 = new RowFilter(CompareOperator.EQUAL, new SubstringComparator("r01999"));
        scan.setFilter(filter3);



        ResultScanner resultScanner = table.getScanner(scan);
        Iterator<Result> results = resultScanner.iterator();

        while (results.hasNext()) {

            Result result = results.next();
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
    }
}

```