## 1，Spark安装

* 01，需要注意一点就是spark安装的时候需要对应版本，如果版本不一致很可能造成一些安装问题
* 02，其他步骤都是类似，下载安装包，解压，然后配置环境变量等等，不再赘述



## 2，Spark的介绍

01，进入Spark命令行，这里是进入本地模式哈

```shell
spark-shell
```

* 本地模式ui地址：http:# master:4040
* 顺便说一下，分布式的是:http:# master:8080



02, sc对象

```shell
srsc是sparkContext，也就是spark上下文对象, spark程序的入口点，封装了整个spark运行环境信息
```



03，RDD

```shell
弹性分布式数据集
```



## 3, Spark使用案例(命令行内)

* 01，wordcount案例， 一句搞定式：
  

```shell
# 第一步读取文件
val rdd01 = sc.textFile("root/test/zhang.txt")
# 第二步按照空格压平，可以直接简化成第二行
# val rdd02 = rdd01.flatMap(line => line.split(" "))
val rdd02 = rdd01.flatMap(_.split(" "))
# 第三步映射
val rdd03 = rdd02.map(word => (word, 1))
# 第四步按key聚合
val rdd04 = rdd03.reduceByKey(_ + _)
# 最后查看结果
rdd04.collect
# res2: Array[(String, Int)] = Array((are,1), (is,2), (man!,1), (You,1), ("",2), (of,2), (ivanl002,1), (world!,2), (ivanl001,1), (awesome,,1), (king,2), (the,2))


# 可以写成如下一行：
val resultRdd = sc.textFile("/root/test/zhang.txt").flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
resultRdd.collect
```



* 02, wordcount加上过滤操作

```shell
val resultRdd = sc.textFile("/root/test/zhang.txt").flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
resultRdd.collect
```



## 4, scala代码编程实现wordcount
*SparkContext:Spark功能的主要入口点。代表到Spark集群的连接，可以创建RDD、累加器和广播变量.每个JVM只能激活一个SparkContext对象，在创建sc之前需要stop掉active的sc。*
  ```scala
package im.ivanl001.bigData.Spark.A01_WordCount

import org.apache.spark.{SparkConf, SparkContext}

// wordcount主程序
object WordCountApp_scala {

  def main(args: Array[String]): Unit = {

    //创建Spark配置对象
    val conf = new SparkConf()
    conf.setAppName("WordCountScala")
    //设置master属性
    conf.setMaster("local") ;

    //通过conf创建sc
    val sc = new SparkContext(conf)

    //加载文本文件
    val rdd1 = sc.textFile(args(0))
    //压扁,可以直接简化成第二行
    //val rdd2 = rdd1.flatMap(line => line.split(" ")) ;
    val rdd02 = rdd01.flatMap(_.split(" "))

    //映射w => (w,1)
    val rdd3 = rdd2.map((_,1))
    val rdd4 = rdd3.reduceByKey(_ + _)
    val r = rdd4.collect()
    //r.foreach(println)
    r.foreach(println(_))
  }
}
  ```

## 5, java代码实现wordcount

```java
package im.ivanl001.bigData.Spark.A01_WordCount;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import scala.Int;
import scala.Tuple2;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-19 19:54
 * #description :
 **/
public class WordCountApp_java {


    public static void main(final String[] args) {

        //1，首先设置配置信息
        SparkConf sparkConf = new SparkConf();
        sparkConf.setAppName("WordCountApp_java");
        sparkConf.setMaster("local");

        //2, 利用配置信息创建spark上下文对象
        JavaSparkContext context = new JavaSparkContext(sparkConf);

        //3，读取文件，获取数据集
        JavaRDD<String> rdd01 = context.textFile("/Users/ivanl001/Desktop/bigData/input/zhang.txt");

        //4, 对获取到到数据集压平处理
        JavaRDD<String> rdd02 = rdd01.flatMap(new FlatMapFunction<String, String>() {
            public Iterator<String> call(String s) throws Exception {
                List<String> flatList = new ArrayList<String>();
                String[] arr = s.split(" ");
                for (String str : arr) {
                    flatList.add(str);
                }
                return flatList.iterator();
            }
        });

        //5，对压平后对数据进行映射处理，方便后续进行计算数量
        JavaPairRDD<String, Integer> rdd03 = rdd02.mapToPair(new PairFunction<String, String, Integer>() {
            public Tuple2<String, Integer> call(String s) throws Exception {
                return new Tuple2<String, Integer>(s, 1);
            }
        });

        //6，对映射后对数据集进行byKey聚合操作
        JavaPairRDD<String, Integer> rdd04 = rdd03.reduceByKey(new Function2<Integer, Integer, Integer>() {
            public Integer call(Integer v1, Integer v2) throws Exception {
                return v1+v2;
            }
        });

        //7, 处理结果，打印出来即可
        List<Tuple2<String, Integer>> result = rdd04.collect();
        for (Tuple2<String, Integer> tuple2 : result) {
            System.out.println("key:" + tuple2._1 + "----and value:"  + tuple2._2);
        }
    }
}
```

> 注意问题,如果报错如下，一般是因为scala编译器版本的问题：
```java
java.lang.NoSuchMethodError: scala.Predef$.refArrayOps([Ljava/lang/Object;)Lscala/collection/mutable/ArrayOps;
```
> 解决办法：
```java
将scala-sdk从2.12换为2.11
File -> Project Structure -> Global libraries -> Remove SDK -> Rebuild.
```