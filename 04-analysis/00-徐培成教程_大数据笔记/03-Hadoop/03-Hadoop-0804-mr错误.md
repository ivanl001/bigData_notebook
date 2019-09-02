1,  这个是DBOutputFormat的一个问题，没找到很好的解决办法

```java
19/08/13 19:13:04 WARN output.FileOutputCommitter: Output Path is null in cleanupJob()
```





数据库输出好像还是有点问题，虽然最后还是输出了数据，但是还是需要设定reducer至少有一个

后面可以研究一下怎么直接输出到数据库，也就是sqoop的底层实现是怎么样子的