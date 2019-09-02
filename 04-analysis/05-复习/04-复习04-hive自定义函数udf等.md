Hive自定义函数包括三种UDF、UDAF、UDTF

　　UDF(User-Defined-Function) 一进一出

　　UDAF(User- Defined Aggregation Funcation) 聚集函数，多进一出。Count/max/min

　　UDTF(User-Defined Table-Generating Functions)  一进多出，如lateral view explore()

　　使用方式 ：在HIVE会话中add 自定义函数的jar文件，然后创建function继而使用函数

**UDF**

1、UDF函数可以直接应用于select语句，对查询结构做格式化处理后，再输出内容。

2、编写UDF函数的时候需要注意一下几点：

　　a）自定义UDF需要继承org.apache.hadoop.hive.ql.UDF。

　　b）需要实现evaluate函数，evaluate函数支持重载。

　　例：写一个返回字符串长度的Demo:



```java
import org.apache.hadoop.hive.ql.exec.UDF;

public class GetLength extends UDF{
    public int evaluate(String str) {
        try{
            return str.length();
        }catch(Exception e){
            return -1;
        }
    }
}
```



3、步骤

　　a）把程序打包放到目标机器上去；

　　b）进入hive客户端，添加jar包：

```shell
hive> add jar /root/hive_udf.jar
```

　　c）创建临时函数：

```shell
hive> create temporary function getLen as 'com.raphael.len.GetLength';
```

　　d）查询HQL语句：

```
hive> select getLen(info) from apachelog;
OK
60
29
87
102
69
60
67
79
66
Time taken: 0.072 seconds, Fetched: 9 row(s)
```



　　e）销毁临时函数：

```shell
hive> DROP TEMPORARY FUNCTION getLen;
```

 

**UDAF**

多行进一行出，如sum()、min()，用在group  by时

1.必须继承

　　org.apache.hadoop.hive.ql.exec.UDAF(函数类继承)

　　org.apache.hadoop.hive.ql.exec.UDAFEvaluator(内部类Evaluator实现UDAFEvaluator接口)

2.Evaluator需要实现 init、iterate、terminatePartial、merge、terminate这几个函数

　　init():类似于构造函数，用于UDAF的初始化

　　iterate():接收传入的参数，并进行内部的轮转，返回boolean

　　terminatePartial():无参数，其为iterate函数轮转结束后，返回轮转数据，类似于hadoop的Combiner

　　merge():接收terminatePartial的返回结果，进行数据merge操作，其返回类型为boolean

　　terminate():返回最终的聚集函数结果

 

　　#开发一个功能同：

　　#Oracle的wm_concat()函数

　　#Mysql的group_concat()

 　　UDAF 详细文档：http://www.cnblogs.com/ggjucheng/archive/2013/02/01/2888051.html

**UDTF**

　　UDTF 详细文档: http://www.cnblogs.com/ggjucheng/archive/2013/02/01/2888819.html