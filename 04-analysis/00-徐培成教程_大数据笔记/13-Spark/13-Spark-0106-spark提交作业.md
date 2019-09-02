## 1, 本地提交

```shell
# 我现在安装模式因为是spark-on-yarn，默认读取hdfs文件
# 运行java版本的
spark2-submit --master local --name IMWordCount  --class im.ivanl001.bigData.Spark.A01_WordCount.a02_WordCount_App01 Spark_Test.jar /user/ivanl001/ivanl001.txt

# 运行scala版本的
spark2-submit --master local --name IMWordCount  --class im.ivanl001.bigData.Spark.A01_WordCount.a01_WordCount_App01 Spark_Test.jar /user/ivanl001/ivanl001.txt
```



