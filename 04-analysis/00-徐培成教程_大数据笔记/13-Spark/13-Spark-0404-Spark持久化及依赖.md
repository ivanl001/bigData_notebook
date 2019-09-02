## 1，持久化

> 01, persist

> 02, cache，这个就是persist的一种，模式是内存模式而已

> 创建持久化： rdd2.persist(StorageLevel.MEMORY_ONLY)

> 删除持久化： rdd2.unpersist()

```
RDD持久化
------------------
	跨操作进行RDD的内存式存储。
	持久化RDD时，节点上的每个分区都会保存操内存中,以备在其他操作中进行重用。
	缓存技术是迭代式计算和交互式查询的重要工具。
	使用persist()和cache()进行rdd的持久化。
	cache()是persist()一种.
	action第一次计算时会发生persist().
	spark的cache是容错的，如果rdd的任何一个分区丢失了，都可以通过最初创建rdd的进行重新计算。
	persist可以使用不同的存储级别进行持久化。


	MEMORY_ONLY			//只在内存
	MEMORY_AND_DISK
	MEMORY_ONLY_SER		//内存存储(串行化)
	MEMORY_AND_DISK_SER 
	DISK_ONLY			//硬盘
	MEMORY_ONLY_2		//带有副本 
	MEMORY_AND_DISK_2	//快速容错。
	OFF_HEAP 

```


* 01, StorageLevel.MEMORY_ONLY
```scala
package im.ivanl001.bigData.Spark.A07_PersistAndCache
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.api.java.JavaSparkContext
import org.apache.spark.storage.StorageLevel
/*
 *
 * 这里演示以下persist的作用，persist虽然是持久化，但是这个持久化可不是持久化到磁盘，persist有模式到说法，可以指定内存模式，磁盘模式等等
 *
 */
object A0701_Persist_Memory {

  def main(args: Array[String]): Unit = {

    //创建Spark配置对象
    val conf = new SparkConf()

    //集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程

    //通过conf创建sc
    val sc = new SparkContext(conf)

    val rdd1 = sc.parallelize(1 to 10)

    val rdd2 = rdd1.map(e => {
      println("ivanl001:" + e)
      e
    })

    //需要先设置缓存才行哈，动作结束之后再设置缓存是无效的
    //这里的cache就是persist的一种，只不过模式设置成了StorageLevel.MEMORY_ONLY
    //rdd2.cache()
    //下面的persist和上面的cache是等价的
    rdd2.persist(StorageLevel.MEMORY_ONLY)

    rdd2.reduce(_ + _)

    //前面已经reduce一次，如果再次reduce的还还会重新重头开始计算，会打印出map中的那些打印的
    //rdd2.reduce(_ + _)

    //但是如果缓存一下就不会重新计算了,但是有一点需要注意一下：缓存需要在第一次动作之前缓存哈
    //rdd2.cache()
    rdd2.reduce(_ + _)
    
    //如果删除持久化，后面的这个聚合还是需要重新transform计算过程的
    rdd2.unpersist()
    rdd2.reduce(_ + _)
  }
}
```

* 01, StorageLevel.DISK_ONLY
```scala
package im.ivanl001.bigData.Spark.A07_PersistAndCache
import org.apache.spark.storage.StorageLevel
import org.apache.spark.{SparkConf, SparkContext}
/*
 *
 * 这里演示以下persist的作用，persist虽然是持久化，但是这个持久化可不是持久化到磁盘，persist有模式到说法，可以指定内存模式，磁盘模式等等
 *
 */
object A0702_Persist_Disk {

  def main(args: Array[String]): Unit = {

    //创建Spark配置对象
    val conf = new SparkConf()

    //集群模式下下面两行不要
    conf.setAppName("WordCountScala")
    //设置master属性
    //conf.setMaster("spark://master:7077")
    conf.setMaster("local[2]")//数字是本地模式下开启几个线程模拟多线程

    //通过conf创建sc
    val sc = new SparkContext(conf)

    val rdd1 = sc.parallelize(1 to 10)

    val rdd2 = rdd1.map(e => {
      println("ivanl001:" + e)
      e
    })

    //需要先设置缓存才行哈，动作结束之后再设置缓存是无效的
    //这里的cache就是persist的一种，只不过模式设置成了StorageLevel.MEMORY_ONLY
    //一般不建议单独磁盘持久化，这样成本会比较高，速度也不会快很多哈
    rdd2.persist(StorageLevel.DISK_ONLY)

    rdd2.reduce(_ + _)

    //前面已经reduce一次，如果再次reduce的还还会重新重头开始计算，会打印出map中的那些打印的
    //rdd2.reduce(_ + _)

    //但是如果缓存一下就不会重新计算了,但是有一点需要注意一下：缓存需要在第一次动作之前缓存哈
    rdd2.reduce(_ + _)
    
    //如果删除持久化，后面的这个聚合还是需要重新transform计算过程的
    rdd2.unpersist()
    rdd2.reduce(_ + _)
  }
}
```

## 2, 宽窄依赖等

* 解释一下：

* NarrowDependency就是那种一对一的依赖关系，比如说我rdd01通过炸裂之后形成一个rdd11,rdd01和rdd11是一一对应的。

* ShuffleDependency则不一样，它是rdd01，rdd02都有可能会进入到rdd11的。

* PruneDependency和上面两种都不一样，rdd11中如果只包含rdd01的一部分，则是PruneDependency

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

```java
创建spark上下文
---------------
[本地模式,通过线程模拟]
	本地后台调度器
	spark local
	spark local[3]							//3线程,模拟cluster集群
	spark local[*]							//匹配cpu个数，
	spark local[3,2]						//3:3个线程，2最多重试次数。


[相当于伪分布式]
	StandaloneSchedulerBackend
	spark local-cluster[N, cores, memory]	//模拟spark集群。

[完全分布式]
	StandaloneSchedulerBackend
	spark spark://s201:7077					//连接到spark集群上.
```