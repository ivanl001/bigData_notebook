第一章：Flink核心概念与实验环境部署

| 章                          | 节                                               | done?               | re-learn? |
| --------------------------- | ------------------------------------------------ | ------------------- | --------- |
| Flink概述                   | 1、阐述Flink的前世今生                           | done✅               | no        |
|                             | 2、Flink生态                                     | done✅               | no        |
|                             | 3、Flink Use Cases                               | done✅               | no        |
|                             | 4、与Hadoop、Spark、Storm等横向对比              | done✅               | no        |
|                             | 5、Flink当前的发展状况以及未来趋势               | done✅               | no        |
| Flink初探                   | 1.Flink 批处理案例实现                           | done✅               | no        |
|                             | 2.Flink 流处理案例实现                           | done✅               | no        |
|                             | 上面这个视频好像也是批处理的内容，没有流处理，过 | 流处理我自己看过了✅ |           |
| Flink核心概念与编程模型(一) | 1、Flink架构                                     | done✅               | no        |
|                             | 2、Stateful Stream Processing                    | done✅               | no        |
|                             | 3、DataStream与DataSet                           | done✅               | no        |
|                             | 4、Table & SQL                                   | done✅               | no        |
|                             | 5、Flink程序基本结构                             | done✅               | no        |
|                             | 6、Flink DataFlow                                | done✅               | no        |
| Flink核心概念与编程模型(二) | 1、window                                        | done✅               | no        |
|                             | 2、Time                                          | done✅               | no        |
|                             | 3、State                                         | done✅               | no        |
|                             | 4、checkpoint与savepoint                         | done✅               | no        |
| Flink Runtime(一)           | 1、Flink 运行时架构                              | done✅               | no        |
|                             | 2、TaskManger Slot                               | done✅               | no        |
|                             | 3、Job Managers, Task Managers, Clients          | done✅               | no        |
|                             | 4、CoLocationGroup                               | done好想不太有印象  | yes       |
|                             | 5、SlotSharingGroup                              | done✅               | yes       |
|                             | 6、Slots && parallelism                          | done✅               | yes       |
|                             | 7、OperatorChain && Task                         | done✅               | yes       |
| Flink Runtime(二)           | 1、Flink 部署方式介绍                            | done✅               | no        |
|                             | 2、Local                                         | done✅               | no        |
|                             | 3、Standalone实验环境部署                        | done✅               | no        |
|                             | 4、Flink On Yarn简述(后续专门章节细讲)           |                     |           |
|                             | 5、Flink job启动方式                             |                     |           |
|                             | 6、Job 的启动过程                                |                     |           |
|                             | 7、Graph                                         |                     |           |
|                             | 8、Flink HA                                      |                     |           |

#### 第二章：Flink DataStream API及项目实战

| 章                          | 节                                     | done?                               | re-learn? |
| --------------------------- | -------------------------------------- | ----------------------------------- | --------- |
| Flink开发环境搭建           | 1、Flink Java开发环境搭建              | done✅                               | no        |
|                             | 2、Flink Scala开发环境搭建             | done✅                               | no        |
|                             | 3、依赖管理                            | done✅                               | yes       |
|                             | 4、Flink源代码编译                     | done✅                               | yes       |
| Flink编程基础               | 1、DataSet与DataStream                 | done✅                               | no        |
|                             | 2、Flink编程基本套路                   | done✅                               | no        |
|                             | 3、Lazy Evaluation                     | done✅                               | no        |
|                             | 4、Specifying Keys                     | done✅                               | yes       |
|                             | 5、Specifying Transformation Functions | done✅                               | yes       |
|                             | 6、Supported Data Types                | done✅7种                            | no        |
|                             | 7、Accumulators & Counters             | done✅没什么内容                     | no        |
|                             | 8、Java Lambda Expressions             | done✅没怎么讲                       | no        |
| DataStreaming API 概述      | 1、再次剖析Streaming示例程序           | done✅                               | no        |
|                             | 2、Graph                               | no这个需要✅                         | yes       |
|                             |                                        |                                     |           |
|                             | 3、DataStreamContext环境               | done✅                               | no        |
|                             | 4、数据源(DataSource)                  | done✅没什么内容                     | no        |
|                             | 5、转化(Transformation)                | done✅主要讲join, connect, project等 | no        |
|                             | 6、数据输出(Sink)                      | done✅没什么内容                     | no        |
|                             | 7、迭代Iterations                      | done✅案例不清晰                     | no        |
|                             | 8、执行参数                            | done✅                               | no        |
|                             | 9、调试                                | done✅                               | no        |
| 状态与容错                  | 1、Flink带状态编程                     | done✅                               | yes       |
|                             | 2、checkpoint与savepoint得细节         |                                     |           |
|                             | 3、broadcast                           |                                     |           |
|                             | 4、State Backends                      |                                     |           |
| Connectors概述              | 1、Connector的概念(Source&Sink)        | done✅                               |           |
|                             | 2、内置Connector                       | done✅                               |           |
|                             | 3、第三方Connector                     | done✅                               |           |
|                             | 4、自定义Connector                     | done✅ soruce，sink                  |           |
| Connectors之Kafka           | Flink与Kafka集成开发实战               | done✅                               | yes       |
| Operators概述               | Operator基本算子介绍                   |                                     |           |
| Operators之Windows操作      | 1、理解Window                          | done✅                               |           |
|                             | 2、理解Time与Watermarks                | done✅可以复习一下                   | yes       |
|                             | 3. Window机制内部实现源码分析          |                                     |           |
|                             | 4. 生产环境中window使用容易遇到的问题  |                                     |           |
| Operators之Join操作         | 1、Window Join                         | done✅                               |           |
|                             | 2、Interval Join                       | done✅                               |           |
| Operators之Process Function | 介绍Low-level算子                      | done✅ processfunction等             | no        |
| Operators之异步IO           | 异步IO方式访问外部数据源               | done✅                               | yes       |
| Side Outputs                | 简单阐述Side Outputs                   |                                     |           |
| Python API                  | 简单阐述一下Python API                 |                                     |           |
| Flink Streaming测试         | 1、Flink Streaming程序单元测试         |                                     |           |
|                             | 2、Flink Streaming程序集成测试         |                                     |           |
| Flink项目实战               | 实时日志分析(上)                       |                                     |           |
|                             | 实时日志分析(中)                       |                                     |           |
|                             | 实时日志分析(下)                       |                                     |           |

#### 第三章：Flink DataSet API及实战

| 章                     | 节                                           | done? | re-learn? |
| ---------------------- | -------------------------------------------- | ----- | --------- |
| DataSet API 概述       | 1、再次剖析DataSet示例程序                   |       |           |
|                        | 2、数据源(DataSource)                        |       |           |
|                        | 3、转化(Transformation)                      |       |           |
|                        | 4、数据输出(Sink)                            |       |           |
|                        | 5、迭代操作                                  | 没看  |           |
|                        | 6、函数中操作数据对象                        |       |           |
|                        | 7、调试                                      |       |           |
|                        | 8、Semantic Annotations                      |       |           |
|                        | 9、广播变量                                  |       |           |
|                        | 10、分布式缓存                               |       |           |
|                        | 11、参数传递                                 |       |           |
| DataSet Transformation | 1、全面介绍各种常见Transformation（1）       |       |           |
|                        | 2、全面介绍各种常见Transformation（2）       |       |           |
| 批处理中的容错机制     | 讲述Flink批处理如何容错                      |       |           |
| 迭代                   | 重点讲述增量迭代                             | 没看  |           |
| Connectors             | 1、内置Connectors                            |       |           |
|                        | 2、HDFS Connector                            |       |           |
| 兼容Hadoop             | 讲述如何跟Hadoop兼容，直接使用Hadoop的MR接口 |       |           |
| DataSetUtils           | 单独讲一下DataSetUtils工具类                 |       |           |
| 基于Flink的ETL项目实践 | 1、需求分析                                  |       |           |
|                        | 2、方案设计                                  |       |           |
|                        | 3、代码实现                                  |       |           |
|                        | 4、运行和优化                                |       |           |
|                        | 5、可视化                                    |       |           |

#### 第四章：Flink扩展学习

| 章                     | 节                                  | done? | re-learn? |
| ---------------------- | ----------------------------------- | ----- | --------- |
| 数据类型与序列化       | 讲述Flink独特的数据类型和序列化方式 |       |           |
| Execution管理          | 1、Execution配置                    |       |           |
|                        | 2、打包                             |       |           |
|                        | 3、并行Execution                    |       |           |
|                        | 4、执行计划                         |       |           |
|                        | 5、Resart策略                       |       |           |
| Flink其他Libraries简介 | 1、CEP简介                          |       |           |
|                        | 2、Storm集成                        |       |           |
|                        | 3、图计算库Gelly简介                |       |           |
|                        | 4、机器学习库简介                   |       |           |
| Flink 最佳实践         | 讲述Flink使用过程中的一些最佳实践   |       |           |

#### 第五章：Flink生产环境与部署

| 章                            | 节                                | done? | re-learn? |
| ----------------------------- | --------------------------------- | ----- | --------- |
| 深入理解Flink On Yarn         | 1、深入讲解Flink On Yarn          |       |           |
|                               | 2、Hadoop集成                     |       |           |
| Flink HA                      | 讲述各种运行模式下的HA            |       |           |
| Flink容错机制大盘点           | 盘点Flink的各种容错机制           |       |           |
| Flink配置                     | 盘点Flink的常见配置               |       |           |
| Flink CLI                     | 讲述Flink CLI操作                 |       |           |
| Flink SSL配置                 | 讲述Flink SSL配置                 |       |           |
| Flink中使用各种分布式文件系统 | 讲述Flink中使用各种分布式文件系统 |       |           |
| Flink应用及版本升级           | 讲述如何升级Flink应用和自身的版本 |       |           |

#### 第六章：Flink Metrics与监控

| 章                            | 节                                        | done? | re-learn? |
| ----------------------------- | ----------------------------------------- | ----- | --------- |
| Flink metrics理解与使用       | 理解metric并讲解其含义                    |       |           |
| Flink日志处理与History Server | 讲解Flink如何处理日志并使用History Server |       |           |
| Flink checkpoint监控          | 讲解检查点的监控                          |       |           |
| Flink延迟反压监控             | 理解反压机制并讲解如何监控                |       |           |
| Flink监控的RESTAPI            | 讲解监控的API                             |       |           |
| Flink app问题剖析诊断         | 结合TSDB讲解如何实现                      |       |           |
|                               | Flink app的问题剖析诊断                   |       |           |