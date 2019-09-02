*UDF也就是user define function，用户自定义函数,之前已经使用了一小点hive自带的函数，如current_user()等*

## 1，一些函数的基本操作
> show functions;
>
>   * 显示函数

> select explode(array(1,2,3));
>
>   * 这个是一个表生成函数, table generate function, 也就是一样生成多行的函数

> describe function concat;
> desc function concat;
>
>   *  显示某个函数的帮助


## 2，自定义函数步骤
* 01， 首先maven中添加依赖 
  ```xml
  <dependency>
      <groupId>org.apache.hive</groupId>
      <artifactId>hive-cli</artifactId>
      <version>2.1.0</version>
  </dependency>
  ```
  
* 02，继承org.apache.hadoop.hive.ql.exec.UDF类，然后添加一个evaluate方法，类似如下:
  ```java
  package im.ivanl001.bigData.Hive;

  import org.apache.hadoop.hive.ql.exec.Description;
  import org.apache.hadoop.hive.ql.exec.UDF;
  
  /**
   * #author      : ivanl001
   * #creator     : 2018-11-06 11:42
   * #description :
   **/
  @Description(name = "imudf_add",value = "value, value, value,xixixixixix", extended = "extended, extended, extended...")
  public class A02_UDF_Add extends UDF {
  
      //这个不是父类的方法，但是需要有这个方法，主要有这种形式即可
      public int evaluate(int a, int b) {
          return a + b;
      }
  
      public int evaluate(int a, int b, int c) {
          return a + b + c;
      }
  }
  ```
  
* 03，达成jar包，放到hive安装目录的lib文件夹下

* o4, 在hive命令下: 把刚才的jar包添加到依赖目录中去，如下命令:
  
> add jar /root/hive/lib/imHive.jar;

* 05, 在hive命令下，给刚才的类添加临时函数方法，方便调用（如果不行，请先退出hive重新进入试一下）
  
> create temporary function add as 'im/ivanl001/bigData/Hive/A02_UDF_Add';

* 06, 一切ok之后，可以查看函数，查看函数描述，查看函数扩展描述等等，也可以正常的调用函数了
  > show functions;
  > desc function add;
  > desc function extended add;
  > select add(1, 3);
  > select add(1, 3, 2);//这两个方法都有evaluate定义，所以都能正常调用

## 3， 自定义日期时间函数

### 01. 第一种方式:继承org.apache.hadoop.hive.ql.exec.UDF类
```java 
package im.ivanl001.bigData.IMUtils;

import org.apache.hadoop.hive.ql.exec.UDF;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-06 15:57
 * #description : 时间处理的自定义函数
 **/
public class IMHiveDateUtils extends UDF {

    //这里需要写下面这个函数，也不是父类的方法，但是写上就可以调用，暂时还不明白是什么原理
    //这里是取出服务器的默认时间，然后转换成字符串，以yyyy-MM-dd HH-mm-ss的格式输出
    public String evaluate() {
        Date current = new Date();
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH-mm-ss");
        String resultStr = simpleDateFormat.format(current);
        return "currentTime:" + resultStr;
    }
}
```

### 02，第二种方式：实现GenericUDF

* 这个应该更高层

* 更加复杂，case when即使用GenericUDF实现的，具体后面再看到可以深入一下

## 4，Hive数据倾斜问题

*教程里面没讲明白，这个好像是处理什么倾斜连接的，也不知道和数据倾斜有啥关系，先放着吧*
```mysql
hive
---------------
	数据倾斜.
	$hive>SET hive.optimize.skewjoin=true;
	$hive>SET hive.skewjoin.key=100000;
	$hive>SET hive.groupby.skewindata=true;


CREATE TABLE mydb.doc(line string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ;

select t.word,count(*) from (select explode(split(line,' ')) word from doc ) t group by t.word ;

```