三级调度：

- DAGScheduler
  - 面向stage
    - shuffleMapStage
    - resultStage
  - DAG调度器会有轮循，不断的提交任务给任务调度器
- TaskScheduler
  - 面向task, 上面的stage就是任务task的集合
  - 每个阶段形成对应的taskSet，提交给后台调度器
- BackendScheduler
  - 使用Nitty框架用rpc实现远程调用
  - 通过Executor执行任务，最终在线程池中不断的执行任务

## 代码流程图如下：最好下载后观看

![13-Spark流程图01.jpg](../%E5%BE%90%E5%9F%B9%E6%88%90%E6%95%99%E7%A8%8B_%E5%A4%A7%E6%95%B0%E6%8D%AE%E7%AC%94%E8%AE%B0/resources/4C832AFE407E66E7F365D4BC63A36398.jpg)

![图画.png](../%E5%BE%90%E5%9F%B9%E6%88%90%E6%95%99%E7%A8%8B_%E5%A4%A7%E6%95%B0%E6%8D%AE%E7%AC%94%E8%AE%B0/resources/B3CC21DEDC58597E06EB6F4069024669.png)



## 2, 宽窄依赖等

- 解释一下：
- NarrowDependency就是那种一对一的依赖关系，比如说我rdd01通过炸裂之后形成一个rdd11,rdd01和rdd11是一一对应的。
- ShuffleDependency则不一样，它是rdd01，rdd02都有可能会进入到rdd11的。
- PruneDependency和上面两种都不一样，rdd11中如果只包含rdd01的一部分，则是PruneDependency

```scala
Dependency:依赖
-------------
	NarrowDependency:	子RDD的每个分区依赖于父RDD的少量分区。
		 |
		/ \
		---
		 |----	OneToOneDependency		//父子RDD之间的分区存在一对一关系。
		 |----	RangeDependency			//父RDD的一个分区范围和子RDD存在一对一关系。

	ShuffleDependency					//依赖，在shuffle阶段输出时的一种依赖。
	
	PruneDependency						//在PartitionPruningRDD和其父RDD之间的依赖
										        //子RDD包含了父RDD的分区子集。
```

## 3, spark模式