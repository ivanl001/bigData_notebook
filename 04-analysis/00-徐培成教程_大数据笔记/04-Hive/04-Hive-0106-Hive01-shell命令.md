# Hive命令

```mysql
# hive中经常设定的，最后一行时情况而定
SET hive.exec.dynamic.partition.mode=nonstrict;
SET hive.exec.max.dynamic.partitions=100000;
SET hive.exec.max.dynamic.partitions.pernode=100000;
SET hive.exec.max.created.files=655350;
SET hive.merge.mapredfiles=TRUE;
# 这个视情况使用，大多时候不设置
SET mapreduce.job.reduces=20;
```



## 1, DDL: 数据库定义: 建表建库

> * 托管表在删除表的时候， 数据也会被删除
>
> - 外部表在删除表的时候， 数据不会被删除

### 1.1, 建库

```mysql
# 创建数据库
create database d1;

# 删除数据库
drop databases d1;
```



### 1.2, 简单建表

```shell
CREATE external TABLE IF NOT EXISTS ivanl001.t1(id int,name string,age int) 
COMMENT '这是注释，注意：如果是托管表，把external去掉即可'; 

# 以下是默认建表的时候使用的参数
# SerDe Library:          org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe       
# InputFormat:            org.apache.hadoop.mapred.TextInputFormat         
# OutputFormat:           org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat
# Compressed:             No 

# row format delimited 
# fields terminated by '\001' 
# collection items terminated by '\002' 
# map keys terminated by '\003'
# lines terminated by '\n' 
# stored as textfile;
```



### 1.3, 指定分隔符和文件格式建表

```shell
# 注意：默认就是文本文件, 默认分隔符是：^A,也就是0x01, 默认换行符是\n
CREATE TABLE IF NOT EXISTS t2(id int,name string,age int) COMMENT '这是注释' ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE;

CREATE external TABLE IF NOT EXISTS t2(id int,name string,age int) COMMENT '这是注释' ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE;
```



1.4, 指定分区

```mysql
CREATE TABLE t5(id int,name string,age int) PARTITIONED BY (Year INT, Month INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ;
```





## 2, DQL：查看命令

### 2.1, 查看描述信息

```mysql
show databases;

use ivanl001;

show tables;

-- 显示某张表的描述信息
desc t1;
desc formatted t1;

-- 显示某个函数的帮助
describe function concat;
desc function concat;
```



### 2.2, 查看建表语句

```mysql
show create table ivanl001.t1;
```



### 2.3, 查看分区

```mysql
show partitions t5;
```





## 3, DML: 操纵语句

### 3.1, 插入语句

```mysql
 insert into ivanl001.t1 values(1, "ivanl001", 1);
```



### 3.2, 加载数据

```mysql
# 注意：如果文件中有分隔符，创建表的时候也必须要添加分隔符限制
load data local inpath '/root/share/customers.txt' overwrite into table t2;

load data inpath '/user/root/customer.txt' into table t3;
```





### 3.3, 复制表

```mysql
# 结构和数据都复制
create table t4_copy as select * from t4;

# 只复制结构
create table t4_struc_copy like t4;
```



### 3.4, 设置表权限

```mysql
# 不允许删除
ALTER TABLE t2 ENABLE NO_DROP;

# 允许删除
ALTER TABLE t2 DISABLE NO_DROP;
```



### 3.5, 更改分区操作

```mysql
alter table t5 add partition (year=2017, month=12);

alter table t5 add partition (year=2018, month=01) 
									 partition(year=2018, month=02);
```



### 3.6, 加载数据到分区

```mysql
load data local inpath '/root/share/customers.txt' into table t5 partition (year=2018, month=02); 
```



## 4, 桶表相关

> 不过在公司里面感觉好像没有使用过桶表

* 桶表的优势：桶表可以和hashmap的分桶类比
* 提高查询性能，速度会快， 比如说16亿数据， 16个桶的话， 查询的时候可以直定位到桶的位置，那么可以直接只从1亿条数据里面查询即可
* 评估数据量，保证每个桶的数据量block的2倍大小。一般也就是256M

```mysql
# 根据id进行hash分桶，把输入的内容分别写出到三个文件中。
create table t6 (id int, name string, age int) clustered by (id) into 3 buckets row format delimited fields terminated by ',';
```



