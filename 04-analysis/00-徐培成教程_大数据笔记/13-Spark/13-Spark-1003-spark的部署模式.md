> 注意一下：部署模式和集群模式是不一样的概念， 集群模式是指服务器是如何配合分布工作的，部署模式概念相对比较小，是指我打包的项目在部署的时候是怎么进行计算。

> 部署模式有两种：
>
> client模式，也就是在哪台服务器上进行部署，这台服务器就是driver，rdd之外的其他运算都是在这台driver上进行, 当然打印也会在这台服务器上打印，方便测试啥的。
>
> cluter模式则不一样， 部署的那台服务器相对而言没什么大的作用，项目会被提交部署到几个worker上去， driver也是由集群找到相对空闲的worker进行充当。这可以避免多个作业提交的时候client模式下服务器压力过大的情况。

## 1, client模式
> spark-submit --class im.ivanl001.bigData.Spark.A12_Spark_DeployMode.A1201_DeployMode_parallelize --master spark://master:7077 --deploy-mode client Spark_Test.jar

## 2, cluster模式

> spark-submit --class im.ivanl001.bigData.Spark.A12_Spark_DeployMode.A1201_DeployMode_parallelize --master spark://master:7077 --deploy-mode cluster Spark_Test.jar