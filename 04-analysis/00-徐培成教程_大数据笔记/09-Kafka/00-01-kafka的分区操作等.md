0, kafka的bin目录中有两个文件可以用来解决重新分区的问题：

> kafka-topics.sh
>
> kafka-reassign-partitions.sh



## 1, 修改分区数

kafka-topic命令实现改变分区数, 注意：只能增加不能减少

```shell
kafka-topics --zookeeper centos01:2181 --alter --topic mytopic --partition 2
```



## 2, 分区再平衡(扩容、删除机器的时候使用)

* 只要配置zookeeper.connect为要加入的集群，再启动Kafka进程，就可以让新的机器加入到Kafka集群。但是新的机器只针对新的Topic才会起作用，在之前就已经存在的Topic的分区，不会自动的分配到新增加的物理机中。为了使新增加的机器可以分担系统压力，必须进行消息数据迁移。Kafka提供了`kafka-reassign-partitions.sh`进行数据迁移。
* 这个脚本提供3个命令：
  - `--generate`: 根据给予的Topic列表和Broker列表生成迁移计划。generate并不会真正进行消息迁移，而是将消息迁移计划计算出来，供execute命令使用。
  - `--execute`: 根据给予的消息迁移计划进行迁移。
  - `--verify`: 检查消息是否已经迁移完成。



topic为`test`目前在broker id为1,2,3的机器上，现又添加了两台机器，broker id为4,5，现在想要将压力平均分散到这5台机器上。

### 2.1, 手动生成一个json文件`topic.json`

* 可选操作

```json
{
"topics": [
{"topic": "test01"}
],
"version": 1
}
```

### 2.2, 调用`--generate`生成迁移计划，将`test`扩充到所有机器上

* 可选操作

```shell
./bin/kafka-reassign-partitions.sh --zookeeper centos01:2181 --topics-to-move-json-file topic.json --broker-list "1,2,3,4,5" --generate
```

生成类似于下方的结果

```json
Current partition replica assignment
{"version":1,"partitions":[{"topic":"test01","partition":1,"replicas":[58,57],"log_dirs":["any","any"]},{"topic":"test01","partition":0,"replicas":[57,58],"log_dirs":["any","any"]}]}

Proposed partition reassignment configuration
{"version":1,"partitions":[{"topic":"test01","partition":1,"replicas":[59,57],"log_dirs":["any","any"]},{"topic":"test01","partition":0,"replicas":[58,59],"log_dirs":["any","any"]}]}
```

`Current partition replica assignment`表示当前的消息存储状况。`Proposed partition reassignment configuration`表示迁移后的消息存储状况。
将迁移后的json存入一个文件`reassignment.json`，供`--execute`命令使用。

所以reassignment.json内容如下：

```json
{"version":1,"partitions":[{"topic":"test01","partition":1,"replicas":[59,57],"log_dirs":["any","any"]},{"topic":"test01","partition":0,"replicas":[58,59],"log_dirs":["any","any"]}]}
```



### 2.3, 执行`--execute`进行扩容。

> 注意：⚠️reassignment.json这个文件可以是用上面两步自动生成的，也可以是自定义的分区，比如说第五台性能好一点，可以多分两个分区也可以，这样就需要自定义比较好一点

```shell
./bin/kafka-reassign-partitions.sh --zookeeper centos01:2181 --reassignment-json-file reassignment.json --execute



# 会打印下面的内容，说明成功了，可以保存打印的内容，方便后续万一需要恢复回到原来的分区
Current partition replica assignment

{"version":1,"partitions":[{"topic":"test01","partition":1,"replicas":[58,57],"log_dirs":["any","any"]},{"topic":"test01","partition":0,"replicas":[57,58],"log_dirs":["any","any"]}]}

Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions.

```

### 2.4, 使用`--verify`查看进度

```shell
./bin/kafka-reassign-partitions.sh --zookeeper centos01:2181 --reassignment-json-file reassignment.json --verify

# 会打印大致如下的结果
Reassignment of partition test01-1 completed successfully
Reassignment of partition test01-0 completed successfully
```



## 3, leader在平衡

* 当一个机器shutdown或者crash的时候，该机器的leader会转交给其他机器，当它重新启动的时候，leader就被其他机器担当了，在某些情况下就会发生leader倾斜的情况
* 可以开启如下的配置：会在该机器启动后重新自动平衡(在cdh中默认开启的哈)
* ![image-20190730151608155](assets/image-20190730151608155.png)





## 相关命令

1. `./bin/kafka-console-producer.sh --broker-list centos01:9092 --topic test`
2. `./bin/kafka-console-consumer.sh --zookeeper centos01:2181 --topic test --from-beginning`
3. `./bin/kafka-topics.sh --zookeeper centos01:2181 --list`
4. `./bin/kafka-topics.sh --zookeeper centos01:2181 --create --replication-factor 2 --partition 6 --topic test`
5. `./bin/kafka-topics.sh --zookeeper centos01:2181 --delete --topic test`
6. `./bin/kafka-topics.sh --zookeeper centos01:2181 --describe --topic test`