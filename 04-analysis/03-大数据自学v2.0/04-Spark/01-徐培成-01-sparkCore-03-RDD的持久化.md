# RDD的持久化

> 默认情况下，RDD的每次action都会从头计算
>
> 如果有需要重用RDD的情况，就可以把RDD持久化到内存或者磁盘(一般不推荐持久化到磁盘)中，然后后续再对这个RDD进行action操作的时候就不需要从头计算，而是直接取出内存中结果即可



## 1, RDD的cache

* cache就是persist的一种，也就是把persist的StorageLevel设置成MEMORY_ONLY就是cache方式了

```java
package im.ivanl001.bigData.Spark.A07_PersistAndCacheAndBroadcast

import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.api.java.JavaSparkContext
import org.apache.spark.storage.StorageLevel

/*
 *
 * 这里演示以下persist的作用，persist虽然是持久化，但是这个持久化可不是持久化到磁盘，persist有模式到说法，可以指定内存模式，磁盘模式等等
 *
 */
object A0701_Persist_cache {

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
    rdd2.cache()
    //下面的persist和上面的cache是等价的
//    rdd2.persist(StorageLevel.MEMORY_ONLY)


    rdd2.reduce(_ + _)

		//如果不设置缓存，会有如下情况：
    //前面已经reduce一次，如果再次reduce的还还会重新重头开始计算，会打印出map中的那些打印的
    //rdd2.reduce(_ + _)

    //但是如果缓存一下就不会重新计算了,但是有一点需要注意一下：缓存需要在第一次动作之前缓存哈，存在下面是❌的
    //rdd2.cache()
    rdd2.reduce(_ + _)

    //如果删除持久化，后面的这个聚合还是需要重新transform计算过程的
    rdd2.unpersist()
    //可以看到上面已经删除持久化了，这里还是会重新进行计算的哦
    rdd2.reduce(_ + _)
  }
}
```





## 2, RDD的persist

### 2.1, persist的存储级别

#### 2.1.1, NONE

* 不持久化

#### 2.1.2, DISK_ONLY

* 只存储到磁盘上

#### 2.1.3, DISK_ONLY_2

* 只存储到磁盘上，但会存储两份，另外一份存储不同的机器上备份一份

#### 2.1.4, MEMORY_ONLY

* 只存储到内存中

#### 2.1.5, MEMORY_ONLY_2

* 只存储到内存中，但会存储两份，另外一份存储在另外的机器上备份

#### 2.1.6, MEMORY_ONLY_SER

* 存储到内存中，并进行序列化减少空间内存占用

#### 2.1.7, MEMORY_ONLY_SER_2

* 存储到内存中，并进行序列化减少空间内存占用，并且会在其他机器上备份一份

#### 2.1.8, MEMORY_AND_DISK

* 内存和磁盘都会存储

#### 2.1.9, MEMORY_AND_DISK_2

* 内存和磁盘都会存储，而且会在另外的机器上备份

#### 2.1.10, MEMORY_AND_DISK_SER

* 内存和磁盘都会存储，而且会进行序列化，减少内存和磁盘空间占用

#### 2.1.11, MEMORY_AND_DISK_SER_2

* 内存和磁盘都会存储，而且会进行序列化，减少内存和磁盘空间占用。并且会在其他机器上做一份相同的备份

#### 2.1.12, OFF_HEAP

* 存储在离堆上



```scala
val NONE = new StorageLevel(false, false, false, false)
val DISK_ONLY = new StorageLevel(true, false, false, false)
val DISK_ONLY_2 = new StorageLevel(true, false, false, false, 2)
val MEMORY_ONLY = new StorageLevel(false, true, false, true)
val MEMORY_ONLY_2 = new StorageLevel(false, true, false, true, 2)
val MEMORY_ONLY_SER = new StorageLevel(false, true, false, false)
val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, false, 2)
val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
val MEMORY_AND_DISK_2 = new StorageLevel(true, true, false, true, 2)
val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false, false)
val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, false, 2)
val OFF_HEAP = new StorageLevel(true, true, true, false, 1)

class StorageLevel private(
  private var _useDisk: Boolean,
  private var _useMemory: Boolean,
  private var _useOffHeap: Boolean,
  private var _deserialized: Boolean,
  private var _replication: Int = 1)
```



## 2.2, persist(newLevel: StorageLevel)的使用

```scala
package im.ivanl001.bigData.Spark.A07_PersistAndCacheAndBroadcast

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
    //保存位置：/private/var/folders/g_/crymrf5n163bqd6c_08lb3h00000gn/T/blockmgr-21f661bf-70df-426c-8fb6-09a14f3269e9
    //如果想要知道保存在哪里，需要关闭log4j的日志相关的信息哈，要不然info打印不出来，也就看不到了


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





## 2.3, unpersist()

* 在持久化的rdd上直接调用unpersist()方法可以删除之前持久化的数据

* 具体使用看上面任何一处代码均可