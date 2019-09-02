## 1, 作业提交部署模式

```shell
--deploy-mode DEPLOY_MODE   Whether to launch the driver program locally ("client") or
                            on one of the worker machines inside the cluster ("cluster")
                            (Default: client).
```



### 01, client模式

* 默认就是client模式

* client模式，也就是在哪台服务器上进行部署，这台服务器就是driver，rdd之外的其他运算都是在这台driver上进行, 当然打印也会在这台服务器上打印，方便测试啥的。

```shell
spark-submit --class im.ivanl001.bigData.Spark.A12_Spark_DeployMode.A1201_DeployMode_parallelize --master spark://master:7077 --deploy-mode client Spark_Test.jar
```



### 02, cluster模式

* cluter模式则不一样， 部署的那台服务器相对而言没什么大的作用，项目会被提交部署到几个worker上去， driver也是由集群找到相对空闲的worker进行充当。这可以避免多个作业提交的时候client模式下服务器压力过大的情况。

```shell
# 这里集群模式，需要每台机器上都要有这个jar包，或者把jar包放在http目录下或者hdfs目录下
spark-submit --class im.ivanl001.bigData.Spark.A12_Spark_DeployMode.A1201_DeployMode_parallelize --master spark://master:7077 --deploy-mode cluster Spark_Test.jar
```



