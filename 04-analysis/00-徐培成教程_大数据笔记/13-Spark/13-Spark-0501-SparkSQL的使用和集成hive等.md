* json文件的读取与写入
* mysql数据库的读取与写入
* hive数据库的集成
```xml
<!--这个依赖下面几个都需要引入-->
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.11</artifactId>
    <version>2.1.0</version>
</dependency>
```

## 1，json文件的读取与写入

```scala
package im.ivanl001.bigData.Spark.A08_SparkSQL

import org.apache.spark.sql.{Row, SaveMode, SparkSession}
import org.apache.spark.{SparkConf, SparkContext}

object A0801_SparkSQL_json {

  def main(args: Array[String]): Unit = {

    //0，首先创建session类，用于后面读取或者写入数据
    val spark = SparkSession
      .builder()
      .appName("Spark SQL basic example")
      .config("spark.some.config.option", "some-value")
      .getOrCreate()

    //1，读取json文件数据
    val df = spark.read.json("/Users/ivanl001/Desktop/bigData/sparkSql/customers.json")
    println(df.show())

    //2, 这个相当于创建表，然后显示出来
    df.createOrReplaceTempView("customers")
    df.show()

    //3，条件查询
    //---------查询年纪大于10的两种方式-----------
    //----------方式一---------------------
    println("方式一")
    val df01 = df.where("age > 10")
    df01.show()

    //4, sql方式查询，查询和3一样的要求内容：age > 10
    //----------方式二------------------
    println("方式二")
    val df02 = spark.sql("select * from customers where age > 10")
    df02.show()
    for (row <- df02) {
      println("ivanl001" + row)
    }

    //5，聚合查询
    //--------聚合查询总数----------
    val count = spark.sql("select count(1) from customers")
    count.show()

    //6, 转换成RDD进行其他操作
    //转成rdd进行处理
    val rdd = df01.rdd
    rdd.foreach(row => {
      val age = row.getLong(0)
      val id = row.getLong(1)
      val name = row.getString(2)
      println("age:" + age + ",name:" + name + ",num:" + id)
    })

    //df01.write.json("/Users/ivanl001/Desktop/bigData/sparkSql/customers01.json")

    //7, 保存
    //设置保存模式
    df01.write.mode(SaveMode.Append).json("/Users/ivanl001/Desktop/bigData/sparkSql/customers01.json")
  }
}
```

## 2，mysql数据库的读取与写入
* 01，需要先引入依赖
  ```xml
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.17</version>
  </dependency>
  ```
* 02, 代码如下 
```java
package im.ivanl001.bigData.Spark.A08_SparkSQL;
import org.apache.spark.sql.Column;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import javax.xml.crypto.Data;
import java.util.Properties;
/**
 * #author      : ivanl001
 * #creator     : 2018-11-30 20:54
 * #description :
 **/
public class A0802_SparkSQL_JDBC_java {

   public static void main(String[] args) {

      //0，创建session对象，方便后面读取或者存入数据库
      SparkSession spark = SparkSession.builder().appName("jdbc").config("spark.master", "local").getOrCreate();
      String url = "jdbc:mysql://localhost:3306/test";
      String table = "customers";

      //1，查询数据库
      Dataset<Row> df = spark.read().format("jdbc").option("url", url).option("dbtable", table).option("user", "root").option("password", "ivanl48265").option("driver", "com.mysql.jdbc.Driver").load();
      df.show();


      //2, 投影查询
      Dataset<Row> df01 = df.select(new Column("name"), new Column("id"));
      df01.show();
      df01.where("id > 2").show();
      //这里可以去重，不过我门这里没有重复的，所以没啥用
      df01.distinct().show();

      //3, 写入到表中
      Properties prop = new Properties();
      prop.put("user", "root");
      prop.put("password", "ivanl48265");
      prop.put("driver", "com.mysql.jdbc.Driver");
      df.write().jdbc(url, "new", prop);
   }
}
```



## 3，hive数据库的集成

* 01，复制hive和hdfs的如下配置文件到spark的conf目录下
*Configuration of Hive is done by placing your hive-site.xml, core-site.xml (for security configuration), and hdfs-site.xml (for HDFS configuration) file in conf/.*
  
  ```shell
  hive-site.xml
  core-site.xml (for security configuration)
  hdfs-site.xml (for HDFS configuration)
  
  
  ```
* 02，复制mysql-connector-java-5.1.17.jar驱动程序到spark到jar目录下
* 03，spark-shell --master local[4]启动程序
* 04，输入如下命令检查是否集成成功
  
  > sql("show databases").show
* 05, 如果显示出数据库列表，集成成功

## 4, spark-hive的使用

* 01，首先在hdfs上传一个用于创建表的数据文件，内容如下:
  
  * /user/ivanl001/sparkSQL_hive_DDL.txt
  
  ```texst
  1,ivanl001,10
  2,ivanl002,20
  3,ivanl003,30
  4,ivanl004,40
  5,ivanl005,50
  6,ivanl006,60
  ```
  
* 02, 在hive上创建表

```SPARQL
spark.sql("create table ivanl001.hiveSpark(id int, name string, age int) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile")
```



* 03, 加载数据


```SPARQL
spark.sql("load data inpath '/user/ivanl001/sparkSQL_hive_DDL.txt' into table imdb.hiveSpark")
```



* 04, 查看数据
  

```SPARQL
sql("select * from imdb.hiveSpark").show
```



* 05, 聚合
  

```SPARQL
sql("select sum(age) from imdb.hiveSpark").show
```



## 5, spark-hive代码流程

* 01,同样拷贝三个配置文件

* 02,因为hive是单机版的，所以如果需要连接需要开启thrift服务器，spark自身已经集成了thrift服务器，可以直接开启spark的thrift服务器，连接spark，通过spark连接hive更为方便，在sbin目录下 
  
  > start-thriftserver.sh --master  spark://master:7077


* 03,如果启动成功，10000端口会被监听
  
> netstat -anop | grep 10000

* 04,通过spark的bin目录下的beeline连接
  
> eeline -u jdbc:hive2://localhost:10000 -d org.apache.hive.jdbc.HiveDriver

* 05,进入命令行后执行查询或者其他任务即可
  
> show databases;

* 06,thrift启动后，maven中添加如下依赖
```xml
<!--spark核心架包-->
<!-- https://mvnrepository.com/artifact/org.apache.spark/spark-core -->
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.11</artifactId>
    <version>2.1.1</version>
</dependency>

<!--sparksql使用的时候需要两个架包-->
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.11</artifactId>
    <version>2.1.0</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.17</version>
</dependency>

<!--sparksql和hive集成需要的架包-->
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
    <version>2.1.0</version>
</dependency>
```

* 07, 编写代码，连接10000端口，进行指定的hive操作即可
```java
package im.ivanl001.bigData.Spark.A09_SparkSQLAndHive;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

/**
 * #author      : ivanl001
 * #creator     : 2018-12-01 12:36
 * #description :
 **/
public class A0901_Spark_Hive_java {

    public static void main(String[] args) throws Exception {

        /*//0，创建session对象，方便后面读取或者存入数据库
        SparkSession spark = SparkSession.builder().appName("jdbc").config("spark.master", "local").getOrCreate();

        //00，尝试创建表
        spark.sql("create table imdb.table(id int)");

        //1，获取
        *//*Dataset<Row> df00 = spark.sql("select * from imdb.hiveSpark");
        df00.show();*/

        //因为如果集成hive，需要开启spark下的beeline和thrift服务器，所以我们这里直接使用10000端口进行连接即可
        //thrift的连接方式也是rpc的，类似mysql，具体如下
        Class.forName("org.apache.hive.jdbc.HiveDriver");
        Connection connection = DriverManager.getConnection("jdbc:hive2://master:10000");
        Statement st = connection.createStatement();

        ResultSet resultSet = st.executeQuery("select * from imdb.hiveSpark");
        while (resultSet.next()) {
            int id = resultSet.getInt("id");
            String name = resultSet.getString("name");
            int age = resultSet.getInt("age");
            System.out.println("id:" + id + "----------aget:" + age + "----------name:" + name);
        }

        ResultSet resultSet1 = st.executeQuery("select count(1) from imdb.hiveSpark");
        while (resultSet1.next()) {
            System.out.printf("个数是："+ resultSet1.getInt("1"));
        }
    }
}
```