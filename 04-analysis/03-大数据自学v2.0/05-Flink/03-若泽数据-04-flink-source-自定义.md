> Sources are where your program reads its input from. You can attach a source to your program by using `StreamExecutionEnvironment.addSource(sourceFunction)`. Flink comes with a number of pre-implemented source functions, but you can always write your own custom sources by implementing the `SourceFunction` for non-parallel sources, or by implementing the `ParallelSourceFunction` interface or extending the `RichParallelSourceFunction` for parallel sources.

自定义数据源的三种方式：

1, 实现SourceFunction接口（非并发）

2, 实现ParallelSourceFunction（并发）

3, 继承RichParallelSourceFunction（并发）

和ParallelSourceFunction类似，但是多了两个方法：open和close方法



其实不太明白一个地方：并发的话， 好像是把同一条消息同时发给多个并发线程？

不是应该把多条消息分发给多个线程吗？？？

是的，上面的猜测是对的， 自定义的打印数字的那个Custom_Source如果是多并行度，肯定会重复的接受到

但是kafka一个topic会有多个分区，然后每个分区一个并行度，就能保证不重复消费，暂时还没有全部理解，但是大概知道意思了