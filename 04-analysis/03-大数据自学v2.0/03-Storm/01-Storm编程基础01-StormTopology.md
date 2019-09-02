#### Storm- Storm Topology-编程基础

##### 1, 基本概念

> storm编程中主要有几个概念

1.1, Storm Topology: storm的拓扑

1.2, Spout: 拓扑的开始，也即是接受数据的地方

1.3, Bolt: 拓扑的关节，是处理数据的地方

1.4, Grouping: 分组, 在Bolt中处理数据的时候，是通过何种方式进行分组



##### 2, 概念具体化，java对象或方法

2.1, StormTopology: 拓扑对象，通过提交拓扑对象进行从而开始任务。拓扑对象中可以设置Spout, Bolt。

```java
// StormTopology的创建方式
TopologyBuilder builder = new TopologyBuilder();
StormTopology stormTopology = builder.createTopology();
```

2.2, IRichSpout: 接口对象，通过这个接口对象接受数据

```java
//IRichSpout的创建方式
继承IRichSpout，实现其方法，然后通过创建子类来创建Spout对象即可
```

2.3, IRichBolt：接口对象，通过这个接口进行数据处理

```java
//IRichBolt的创建方式
继承IRichBolt，实现其方法，然后通过创建子类来创建Bolt对象即可
```

2.4, Grouping: 这个其实是方法，最常用的有：shuffleGrouping：随机分区，fieldsGrouping：字段分区，allGrouping：广播分区，directGrouping：指定特定分区。当然也可以实现CustomStreamGrouping接口，自定义分组方式。

```java
//自带的Grouping使用方式
builder.setBolt("bolt01", new WordCountSplitBolt()).shuffleGrouping("spout");
//自定义的Grouping使用方式
builder.setBolt("bolt01", new WordCountSplitBolt()).customGrouping("spout", new IMCustomGrouping());
```



2.5, 具体的使用方法：

```java
//0，创建配置文件，配置相应的内容
Config config = new Config();
config.setDebug(false);

//1，创建拓扑建设者设对象
TopologyBuilder builder = new TopologyBuilder();

//2，拓扑建设者设对象设置读数据的spout
builder.setSpout("call-log-reader-spout", new A01_Storm01_Spout());

//3，拓扑建设者设对象设置第一个计算节点Bolt，A01_Storm02_Bolt01集成IRichSpout
builder.setBolt("call-log-creator-bolt", new A01_Storm02_Bolt01()).shuffleGrouping("call-log-reader-spout");

//4，拓扑建设者设对象设置第二个计算节点Bolt，A01_Storm02_Bolt02集成IRichSpout
builder.setBolt("call-log-counter-bolt", new A01_Storm02_Bolt02()) .fieldsGrouping("call-log-creator-bolt", new Fields("call"));

//5，拓扑建设者设对象建设拓扑，然后提交拓扑开始作业
StormTopology stormTopology = builder.createTopology();
StormSubmitter.submitTopology("CallLogAnalyserStorm", config, builder.createTopology());
```

