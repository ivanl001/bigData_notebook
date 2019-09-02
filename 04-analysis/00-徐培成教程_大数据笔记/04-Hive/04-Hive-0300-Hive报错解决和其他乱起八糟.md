```java
FAILED: SemanticException Unable to determine if hdfs://master/user/hive/warehouse/imdb.db/t9 is encrypted: java.lang.IllegalArgumentException: Wrong FS: hdfs://master/user/hive/warehouse/imdb.db/t9, expected: hdfs://imcluster
```

*这个错误是因为hdfs由原来的分布式更改为高可用模式后，hive原先在分布式下创建的表中某些参数没有更改，也就是表的目录位置还是原先的master而不是现在高可用模式下的imcluster导致的*

解决方案(mysql中执行如下操作):

* 1，更改DBS表中的DB_LOCATION_URI里的master为imcluster
  
> select DB_LOCATION_URI from DBS
  
* 2，更改SDS中的LOCATION里的master为imcluster
  
> select LOCATION from SDS
  
* 3，重新查询该表应该ok了

## 01，一些碎知识

---
注：union在mysql和hive中使用方法一样
* union查询的使用：
* mysql> select id, orderno from orders union select id, name from customers;
* 这个的作用是把不同表查出来的内容竖直的排列在一起， 跟join水平排列在一起是对应的。但是需要注意：union前后的列数,也就是字段个数需要一致，要不然没法排列在一起。
* 单机版的hive命令行启动后也会有一个runJar的线程哈
---

---
*这里顺便先介绍一下打包的两种方式吧：*
* 1，maven打包， 直接在pom文件中添加：然后通过右侧的maven窗口的package双击即可   
```xml
<packaging>jar</packaging>
<build>
    <!--这里可以自定义打包的文件的名字哈-->
    <finalName>imHive</finalName>
</build>
```
* 2, java默认打包
  > jar cvf IMHive.jar UDF_Add.java
    * 这个是只打印一个文件
  > jar cvf IMHive.jar -C class/ .
    * 这个是打印class文件夹下的所有文件，注意：这里class文件夹是编译后的class文件的文件夹哈，别搞错了，如果打java文件，是没啥作用的
---