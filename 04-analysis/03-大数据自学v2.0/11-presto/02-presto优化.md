## 1 Presto优化之数据存储

### 1.1 合理设置分区

与Hive类似，Presto会根据元数据信息读取分区数据，合理的分区能减少Presto数据读取量，提升查询性能。

### 1.2 使用列式存储

Presto对ORC文件读取做了特定优化，因此在Hive中创建Presto使用的表时，建议采用ORC格式存储。相对于Parquet，Presto对ORC支持更好。

### 1.3 使用压缩

数据压缩可以减少节点间数据传输对IOSnappy



> 存储用orc，压缩用snappy



## 2 Presto优化之查询SQL

### 2.1 只选择使用的字段

由于采用列式存储，选择需要的字段可加快字段的读取、减少数据量。避免采用*读取所有字段。

```sql
[GOOD]: SELECT time, user, host FROM tbl

[BAD]:  SELECT * FROM tbl
```



### 2.2 过滤条件必须加上分区字段 

对于有分区的表，where语句中优先使用分区字段进行过滤。acct_day是分区字段，visit_time是具体访问时间。

```sql
[GOOD]: SELECT time, user, host FROM tbl where acct_day=20171101

[BAD]:  SELECT * FROM tbl where visit_time=20171101
```



### 2.3 Group By语句优化 

合理安排Group by语句中字段顺序对性能有一定提升。将Group By语句中字段按照每个字段distinct数据多少进行降序排列。

> 将Group By语句中字段按照每个字段distinct数据多少进行降序排列。

```sql
[GOOD]: SELECT GROUP BY uid, gender

[BAD]:  SELECT GROUP BY gender, uid
```



### 2.4 Order by时使用Limit

Order by需要扫描数据到单个worker节点进行排序，导致单个worker需要大量内存。如果是查询Top N或者Bottom N，使用limit可减少排序计算和内存压力。

```sql
[GOOD]: SELECT * FROM tbl ORDER BY time LIMIT 100

[BAD]:  SELECT * FROM tbl ORDER BY time
```



### 2.5 使用Join语句时将大表放在左边

Presto中join的默认算法是broadcast join，即将join左边的表分割到多个worker，然后将join右边的表数据整个复制一份发送到每个worker进行计算。如果右边的表数据量太大，则可能会报内存溢出错误。

```sql
[GOOD] SELECT ... FROM large_table l join small_table s on l.id = s.id

[BAD] SELECT ... FROM small_table s join large_table l on l.id = s.id
```



## 3 注意事项

### 3.1 字段名引用

避免和关键字冲突：MySQL对字段加反引号**`**、Presto对字段加双引号分割

> Presto对字段加双引号分割

当然，如果字段名称不是关键字，可以不加这个双引号。

### 3.2 时间函数

对于Timestamp，需要进行比较的时候，需要添加Timestamp关键字，而MySQL中对Timestamp可以直接进行比较。

```sql
/MySQL的写法/
SELECT t FROM a WHERE t > '2017-01-01 00:00:00'; 

/Presto中的写法/
SELECT t FROM a WHERE t > timestamp '2017-01-01 00:00:00';
```



### 3.3 不支持INSERT OVERWRITE语法

Presto中不支持insert overwrite语法，只能先delete，然后insert into。

### 3.4 PARQUET格式

Presto目前支持Parquet格式，支持查询，但不支持insert。