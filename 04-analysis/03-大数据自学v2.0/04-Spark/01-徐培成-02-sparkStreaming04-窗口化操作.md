# Window Operations

http://spark.apache.org/docs/2.1.0/streaming-programming-guide.html#window-operations



## 1, 窗口分类

### 01, 窗口类型分类

时间窗口

事件窗口



### 02, 不同操作分类

滑动窗口

滚动窗口







2, 窗口操作

Some of the common window operations are as follows. All of these operations take the two parameters - *windowLength* and *slideInterval*.

| Transformation                                               | Meaning                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| **window**(*windowLength*, *slideInterval*)                  | Return a new DStream which is computed based on windowed batches of the source DStream. |
| **countByWindow**(*windowLength*, *slideInterval*)           | Return a sliding window count of elements in the stream.     |
| **reduceByWindow**(*func*, *windowLength*, *slideInterval*)  | Return a new single-element stream, created by aggregating elements in the stream over a sliding interval using *func*. The function should be associative and commutative so that it can be computed correctly in parallel. |
| **reduceByKeyAndWindow**(*func*, *windowLength*, *slideInterval*, [*numTasks*]) | When called on a DStream of (K, V) pairs, returns a new DStream of (K, V) pairs where the values for each key are aggregated using the given reduce function *func* over batches in a sliding window. **Note:** By default, this uses Spark's default number of parallel tasks (2 for local mode, and in cluster mode the number is determined by the config property `spark.default.parallelism`) to do the grouping. You can pass an optional `numTasks` argument to set a different number of tasks. |
| **reduceByKeyAndWindow**(*func*, *invFunc*, *windowLength*, *slideInterval*, [*numTasks*]) | A more efficient version of the above `reduceByKeyAndWindow()` where the reduce value of each window is calculated incrementally using the reduce values of the previous window. This is done by reducing the new data that enters the sliding window, and “inverse reducing” the old data that leaves the window. An example would be that of “adding” and “subtracting” counts of keys as the window slides. However, it is applicable only to “invertible reduce functions”, that is, those reduce functions which have a corresponding “inverse reduce” function (taken as parameter *invFunc*). Like in `reduceByKeyAndWindow`, the number of reduce tasks is configurable through an optional argument. Note that [checkpointing](http://spark.apache.org/docs/2.1.0/streaming-programming-guide.html#checkpointing) must be enabled for using this operation. |
| **countByValueAndWindow**(*windowLength*,*slideInterval*, [*numTasks*]) | When called on a DStream of (K, V) pairs, returns a new DStream of (K, Long) pairs where the value of each key is its frequency within a sliding window. Like in `reduceByKeyAndWindow`, the number of reduce tasks is configurable through an optional argument. |
|                                                              |                                                              |

#### Join Operations