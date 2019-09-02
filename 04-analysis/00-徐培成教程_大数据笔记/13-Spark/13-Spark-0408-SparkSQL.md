## 通过命令行使用SparkSQL

> case class Customer (id:Int, name:String, age:Int)

> val arr = Array("1,ivanl001,12", "2,ivanl002,22", "3,ivanl003,33")

> val rdd1 = sc.makeRDD(arr)

> val rdd2 = rdd1.map(e => {
      val arr = e.split(",")
      new Customer(arr(0).toInt, arr(1), arr(2).toInt)
    })
    
> val df = spark.createDataFrame(rdd2)    //相当于创建表

> df.printSchema   //显示表结构

> df.createTempView("customer")  //这里类似创建临时表，后面sql语句用到

> df.show  //显示内 容

> val rdd000 = spark.sql("select * from customer")
> rdd000.show   //显示刚才sql语句的结果

> val rdd000 = spark.sql("select * from customer where age > 15")
> rdd000.show

//当然上面的有条件的rdd也还可以再注册成新的小表，再进行其他的操作等等

> val count = spark.sql("select count(1) from customer");
> count.show  //这里的结果其实都是rdd，通过show显示内容,错了，不是rdd，是dataframe

> val lim = spark.sql("select * from customer limit 2")
> lim.show

## 半SQL
> spark.sql("select id,name from customer").show
> df.selectExpr("id", "name").show

> df.where("name like 'ivanl%'").show

> df.agg(sum("age")).show            //这个是得到一个dataframe，然后显示
> df.agg(sum("age"), max("age"), min("age")).show
> df.map(_.getAs[Int]("age")).reduce(_+_)  //算所有年龄的总和



