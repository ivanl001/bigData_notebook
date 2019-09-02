集成比较简单：
* 01，复制hadoop的core-site.xml和hdfs-site.xml到spark的conf目录下
* 02, 注意分发到所有到spark节点上
* 03, 启动spark
  > spark-shell --master spark://master:7077
* 04, 通过如下命令连接hdfs集群
  > sc.textFile("hdfs://imcluster/user/root/test/test01.txt").collect();

### 尝试用一行代码搞定数据倾斜的spark问题：

> sc.textFile("hdfs://imcluster:/user/root/zhang.txt",4).flatMap(_.split(" ")).map(word => {import scala.util.Random;(word + "_" + Random.nextInt(4), 1)}).reduceByKey(_ + _).map(e => {(e._1.split("_")(0),e._2)}).reduceByKey(_ + _).collect()