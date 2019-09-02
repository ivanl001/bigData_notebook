# SparkSQL

```xml
<!--这个依赖下面几个都需要引入-->
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.11</artifactId>
    <version>2.1.0</version>
</dependency>
```

- 1, sparkSQL基本操作
  - 这里是用自己造一个数组转成rdd，再转成dataframe进行测试
- 2, sparkSQL处理json
  - 这里处理json文件格式，但确切说是不带最外的[]的json串
  - 而且保存的时候，是逗号也不会保存。
- 3, mysql数据库的读取与写入
- 4, hive数据库的集成
  - 集成hive放在后面一个内容去讲



## 1, sparkSQL基本操作

### 1.0, 代码

```scala
package im.ivanl001.bigData.Spark.A08_SparkSQL

import org.apache.spark.sql.SparkSession
import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-18 15:33
  * #description : 通过自定义数组转换成对象rdd，然后创建dataframe，然后注册表，然后进行表查询语句
  *
  **/
object A0801_SparkSQL_begin {

  case class Person(id:Int, name:String, age:Int)

  def main(args: Array[String]): Unit = {

    //0, 首先设置配置，并根据配置创建sc
    val conf = new SparkConf()
    conf.setAppName("sql_begin")
    conf.setMaster("local[2]")
    val sc = new SparkContext(conf)

    //1，首先创建session类，用于后面读取或者写入数据
    val spark = SparkSession
      .builder()
      .appName("Spark SQL basic example")
      .config("spark.master", "local")
      .getOrCreate()


    //2, 创建数据源
    val arr = Array("1 tom 12", "2 tomas 123", "3 tomasLee 312")
    val rdd01 = sc.parallelize(arr)
    rdd01.foreach(println(_))

    //3, 根据rdd创建
    val rdd02 = rdd01.map(str => {
      val arr = str.split(" ")
      Person(arr(0).toInt, arr(1), arr(2).toInt)
    })

    //4, 根据对象的rdd创建dataframe
    val df01 = spark.createDataFrame(rdd02)

    //5, 注册表
    df01.createOrReplaceTempView("customer")

    //6, 打印表信息
    //这里没有返回值哦
    df01.printSchema()

    //7, 从表中选择需要的数据
    val rdd_customer_01 = spark.sql("select * from customer")
    rdd_customer_01.show()
  }
}
```



## 2, sparkSQL处理json

> 明确来说，这里处理的json要求如果是数组，需要把最外层的[]去掉，具体看代码
>
> 而且保存的时候，是逗号也不会保存。

### 2.0, 代码

```scala
package im.ivanl001.bigData.Spark.A08_SparkSQL

import org.apache.spark.sql.{Row, SaveMode, SparkSession}
import org.apache.spark.{SparkConf, SparkContext}

/**
  * #author      : ivanl001
  * #creator     : 2019-08-18 15:33
  * #description : 准确来说这个不是读取json，因为json数组的是需要外面加[],但是这里要求不能加[].如下
  *
  * 这种数据是不能正确读取的
  * [{"id":1, "name":"tom", "age":11},
    {"id":2, "name":"tomas", "age":12},
    {"id":3, "name":"tomasLee", "age":13}]
  *
  * 需要把外面的[]给去掉
  * 下面才能正确读取
  * {"id":1, "name":"tom", "age":11},
    {"id":2, "name":"tomas", "age":12},
    {"id":3, "name":"tomasLee", "age":13}
  *
  **/

object A0802_SparkSQL_json {

  def main(args: Array[String]): Unit = {

    //0，首先创建session类，用于后面读取或者写入数据
    val spark = SparkSession
      .builder()
      .appName("Spark SQL basic example")
      .config("spark.master", "local")
      .getOrCreate()

    //1，读取json文件数据
    val df = spark.read.json("/Users/ivanl001/Desktop/00-bigData/00-data/input/customers.json")
    //打印表相关信息
    //df.printSchema()
    //打印内容
    //df.show()

    //2, 这个相当于创建表，然后显示出来
    df.createOrReplaceTempView("customers")
    //df.show()


    //3，条件查询
    //---------查询年纪大于10的两种方式-----------
    //----------方式一---------------------
    println("方式一")
    val df01 = df.where("age > 11")
    //df01.show()


    //4, sql方式查询，查询和3一样的要求内容：age > 10
    //----------方式二------------------
    println("方式二")
    val df02 = spark.sql("select * from customers where age > 12")
    //df02.show()
    /*for (row <- df02) {
      println("ivanl001" + row)
    }*/

    //5，聚合查询
    //--------聚合查询总数----------
    val count = spark.sql("select count(1) from customers")
    /*count.show()
    for (row <- count) {
      //这里可以直接打印出总数，一共是一行哈
      println("ivanl002" + row)
    }*/

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
    df01.write.mode(SaveMode.Append).json("/Users/ivanl001/Desktop/00-bigData/00-data/output/sparkSql_json/customers01.json")
  }
}
```



## 3, mysql数据库的读取与写入

### 3.1, 需要先引入依赖

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.17</version>
</dependency>
```

###  3.2, 代码

```java
package im.ivanl001.bigData.Spark.A08_SparkSQL;

import org.apache.spark.sql.*;

import javax.xml.crypto.Data;
import java.util.Properties;
import java.util.function.Consumer;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-30 20:54
 * #description :
 **/
public class A0803_SparkSQL_JDBC_java {

   public static void main(String[] args) {

      //0，创建session对象，方便后面读取或者存入数据库
      SparkSession spark = SparkSession.builder().appName("jdbc").config("spark.master", "local").getOrCreate();
      String url = "jdbc:mysql://centos01:3306/test";
      String table = "users";


      //1，查询数据库
      Dataset<Row> df = spark.read().format("jdbc").option("url", url).option("dbtable", table).option("user", "root").option("password", ",.").option("driver", "com.mysql.jdbc.Driver").load();
      df.createOrReplaceTempView("users");
      Dataset<Row> df01 = spark.sql("select email,psd from users");
      //show默认情况下显示20行，可以在括号中设置需要显示的行数
      //df01.show(100);


      //转换成rdd后应该能显示出所有的数据
      df01.toJavaRDD().collect().forEach(new Consumer<Row>() {
         @Override
         public void accept(Row row) {
            String email = row.getString(0);
            String psd = row.getString(1);
            System.out.println("id:" + email + ",age:" + psd) ;
         }
      });


      //2, 投影查询
      /*Dataset<Row> df01 = df.select(new Column("name"), new Column("id"));
      df01.show();
      df01.where("id > 2").show();
      //这里可以去重，不过我门这里没有重复的，所以没啥用
      df01.distinct().show();

      //3, 转换成rdd进行操作
      /*Dataset<Row> df01 = spark.sql("select id, name, age from customers");
      df01.show();

      df01.toJavaRDD().collect().forEach(new Consumer<Row>() {
         @Override
         public void accept(Row row) {
            long age = row.getInt(0);
            String name = row.getString(1);
            long id = row.getInt(2);
            System.out.println("id:" + id + ",age:" + age + ",name: " + name) ;
         }
      });*/

      //4, 写入到表中
      Properties prop = new Properties();
      prop.put("user", "root");
      prop.put("password", ",.");
      prop.put("driver", "com.mysql.jdbc.Driver");
      //url决定具体写到哪个地方的哪个库
      df.write().mode(SaveMode.Append).jdbc(url, "new", prop);
      //df.write().jdbc(url, "new", prop);
   }
}
```



## 4, hive数据库的集成

* 看后面一片问斩