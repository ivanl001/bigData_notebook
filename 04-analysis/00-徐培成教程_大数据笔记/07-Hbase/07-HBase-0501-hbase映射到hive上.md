*详细内容可以参考书籍《O'REILLY]Programming Hive.pdf》*

## 1，创建一个新的hive表，用hbase存储

```sql
CREATE TABLE hbase_stocks(key INT, name STRING, price FLOAT) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,stock:val") TBLPROPERTIES ("hbase.table.name" = "stocks");
```

## 2, 映射已经存在的hbase表到hive中

```sql
<!--在hbase中默认是存放在default中的哈，当然也可以按照第二个方式直接映射到指定命名空间的表中-->
CREATE TABLE t00_hive_hbase(key int, name string)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf:name") TBLPROPERTIES("hbase.table.name" = "t00_hive_hbase");

<!--映射到指定的表-->
CREATE TABLE t01_hive_hbase(key int, name string)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf:name") TBLPROPERTIES("hbase.table.name" = "ns1:t01_hive_hbase");
```
*上面是书中给出的，我们这里把我们之前的*



测试映射, :key代表是rowkey， 其余的需要一一对应才行，对应不上自动填充null

> CREATE external TABLE t1_hbase_hive (key string, c1 string, c2 string, c3 string) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
> WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key, f1:c1, f1:c2, f1:c3") TBLPROPERTIES("hbase.table.name" = "ns1:t1");

