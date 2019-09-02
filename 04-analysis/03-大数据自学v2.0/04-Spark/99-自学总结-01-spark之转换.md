## 1, RDD  to DF

* 注意：RDD转换成DF需要隐式类型转换

```scala
import spark.implicits._
```



## 2, join

只有RDD[(K, W)]这中key-value类型的RDD才能进行join

```scala
rdd01_03.join(rdd02_02).map{
  case (userId, (sessionInfo, userInfo)) => {
    val age = userInfo.age
    val pro = userInfo.professional
    val sex = userInfo.sex
    val city = userInfo.city

    val fullInfo = sessionInfo + "|" +
    Constants.FIELD_AGE + "=" + age + "|" +
    Constants.FIELD_PROFESSIONAL + "=" + pro + "|" +
    Constants.FIELD_SEX + "=" + sex + "|" +
    Constants.FIELD_CITY + "=" + city + "|"
    val sessionId = StringUtils.getFieldFromConcatString(sessionInfo, "\\|", Constants.FIELD_SESSION_ID)
    (sessionId, fullInfo)
  }
}
```



但是对于DataFrame（DF）而言，彼此两两都可以join，但是join的时候需要指定column

```scala
// im.ivanl001.A07_ItemCF_Recommender

//在原有的评分表中添加count字段
//这里就得到了每个商品总的评分次数
val ratingWithCount = ratingDF.join(productRatingCountDF, "productId")
//ratingWithCount.show()
```

