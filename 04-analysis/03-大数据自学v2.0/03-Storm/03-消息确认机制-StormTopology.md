1，IRichBolt消息处理完进行回调

消息处理完之后，通过

* 调用OutputCollector的ack()方法进行成功消息回调
* 调用OutputCollector的fail()方法进行失败消息回调



2，接受到回调进行处理

如果Bolt中有回调消息

* 成功ack()回调会激活方法：

  ```java
  public void ack(Object timstamp) {
    //成功回调方法
  }
  ```

* 失败fail()回调会激活方法：

  ```java
  public void fail(Object timstamp) {
    //失败回调方法
  }
  ```

  

——— 其他一些

0，storm属于无状态的实时流计算，能保证at lease once,但是吞吐量应该比flink要小，单独部署，不需要依赖yarn等，适合小的实时程序。

1,  数据分组方式方式和mr的分区类似，storm的wordcount案例中是用fieldsGrouping的，但是这样会造成数据倾斜，解决办法也和mr类似，

- 用shuffleGrouping，也就是直接混洗后计算出数量，然后再用fieldsGrouping把相同的单词分区到一起，用两次聚合避免倾斜

