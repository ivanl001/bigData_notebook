## 4, spark中hive使用



### 4.1,集成hive

> Spark SQL also supports reading and writing data stored in [Apache Hive](http://hive.apache.org/). However, since Hive has a large number of dependencies, these dependencies are not included in the default Spark distribution. If Hive dependencies can be found on the classpath, Spark will load them automatically. Note that these Hive dependencies must also be present on all of the worker nodes, as they will need access to the Hive serialization and deserialization libraries (SerDes) in order to access data stored in Hive.
>
> Configuration of Hive is done by placing your `hive-site.xml`, `core-site.xml` (for security configuration), and `hdfs-site.xml` (for HDFS configuration) file in `conf/`.
>
> When working with Hive, one must instantiate `SparkSession` with Hive support, including connectivity to a persistent Hive metastore, support for Hive serdes, and Hive user-defined functions. Users who do not have an existing Hive deployment can still enable Hive support. When not configured by the `hive-site.xml`, the context automatically creates `metastore_db` in the current directory and creates a directory configured by `spark.sql.warehouse.dir`, which defaults to the directory `spark-warehouse` in the current directory that the Spark application is started. Note that the `hive.metastore.warehouse.dir` property in `hive-site.xml` is deprecated since Spark 2.0.0. Instead, use `spark.sql.warehouse.dir` to specify the default location of database in warehouse. You may need to grant write privilege to the user who starts the Spark application.
>
> 
>
> 上面的大概意思就是：默认情况下，如果spark找不到hive的配置文件hive-site.xml，会在启动的时候报错：
>
> 然后会在启动的目录下创建一个文件夹metastore_db文件夹作为hive的数据库。
>
> 那么配置的方法就有两种：
>
> 第一种：把hive的hive-site.xml放置到spark的配置文件夹目录conf下
>
> 第二种：在hive-site.xml中配置参数spark.sql.warehouse.dir为metastore_db的文件夹的位置，这种方式下，spark的数据库和hive的数据库会分开，具体内容参考官网：http://spark.apache.org/docs/2.1.0/sql-programming-guide.html#hive-tables
>
> 我本人使用的cdh版本的spark2.1.0, 然后spark2.1.0的History Server安装在centos01上，所以一般也是用centos01的shell登陆spark2-shell的。所以我使用第一种方式把hive的配置文件放置在spark的配置目录下。
>
> 因为centos02，centos03上没有History Server，所以也没有/etc/spark2/conf.cloudera.spark2_on_yarn/这个配置文件夹，所以暂时只给centos01的shell添加hive支持，在centos01上登陆shell即可



#### 01, 添加hive的配置文件链接

* 在spark2.1.0的History Server的安装机器上给spark2.1.0的配置中添加hive-site.xml 的配置链接

```shell
ln -s /etc/hive/conf.cloudera.hive/hive-site.xml /etc/spark2/conf.cloudera.spark2_on_yarn/hive-site.xml

ln -s /etc/hive/conf.cloudera.hive/hive-site.xml /etc/spark2/conf/hive-site.xml
```



#### 02, spark2-shell登陆验证

```shell
spark2-shell
# 如下命令如果能正确打印出hive的数据库，说明配置已经ok
sql("show databases").show
```

### 4.2, spark2-shell中hive的使用

#### 00, 准备数据

* 首先在hdfs上传一个用于创建表的数据文件，内容如下:

- /user/ivanl001/sparkSQL_hive_DDL.txt

```texst
1,ivanl001,10
2,ivanl002,20
3,ivanl003,30
4,ivanl004,40
5,ivanl005,50
6,ivanl006,60
```

#### 01, 在hive上创建表

```SPARQL
spark.sql("create table ivanl001.hiveSpark(id int, name string, age int) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile")
```



#### 02, 加载数据

```SPARQL
spark.sql("load data inpath '/user/ivanl001/sparkSQL_hive_DDL.txt' into table ivanl001.hiveSpark")
```



#### 03, 查看数据

```SPARQL
sql("select * from ivanl001.hiveSpark").show
```



#### 04, 聚合

```SPARQL
sql("select sum(age) from ivanl001.hiveSpark").show
```



### 4.3, spark代码中hive的使用

```java
package im.ivanl001.bigData.Spark.A08_SparkSQL

import java.io.File

import org.apache.spark.sql.{Dataset, Row, SparkSession}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-18 17:04
  * #description : 使用sparkSQL来定义hive表等
  *
  **/
object A0804_SparkSQL_hive_DDL {

  def main(args: Array[String]): Unit = {

    //不管用，暂时不知道怎么在spark中操纵hive
    /*val warehouseLocation = new File("spark-warehouse").getAbsolutePath
    println(warehouseLocation)*/


    // warehouseLocation points to the default location for managed databases and tables
//    val warehouseLocation = "/user/hive/warehouse"

    //0, 注意：如果是集成已经存在的hive表，需要把hive-site.xml配置文件放到资源目录下哦
    //1，创建SparkSession
    val spark = SparkSession
      .builder()
      .appName("Spark Hive Example")
      .config("spark.master", "local")
//      .config("spark.sql.warehouse.dir", warehouseLocation)
      .enableHiveSupport()
      .getOrCreate()

    //2，sql功能演示：显示数据库
    spark.sql("show databases").show()

    //3，读取hiv表，并为hive表注册spark表，方便后面读取
    val ds00 = spark.read.table("ivanl001.hiveSpark")
    ds00.createOrReplaceTempView("t1")

    val ds01 = spark.sql("select * from t1")
    ds01.show()

    //4，把刚才读取的表转成rdd并进行解析
    val rdd01 = ds01.rdd
    rdd01.collect().foreach(row => {
      println("id:"+ row.get(0))
      println("name:"+ row.get(1))
      println("age:"+ row.get(2))
    })
  }
}
```

