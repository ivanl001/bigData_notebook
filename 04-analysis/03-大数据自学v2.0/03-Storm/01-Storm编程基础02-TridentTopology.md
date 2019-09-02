#### Storm- TridentTopology-编程基础



##### 1, 基本概念

> TridentTopology编程中主要有几个概念

1.1, **TridentTopology**: storm的高级拓扑, 是对StormTopology拓扑的上层封装

* 可以将`StormTopology`与`TridentTopology`的关系，类比为JDBC与ORM框架(mybatis、hibernate)之间的关系，后者是前者的高级封装，功能相同，但是可以极大的减少我们的开发的工作量。

1.2, **Trident Spout**: 拓扑的开始，也即是接受数据的地方

1.3, **Operation**: 拓扑的操作，是处理数据的地方，是针对Bolt、Grouping等概念的抽象

* 

1.4, **State**是新提出的概念，实际上就是数据持久化的接口



##### 2, 概念具体化，java对象或方法

2.1, TridentTopology: 拓扑对象，通过提交拓扑对象进行从而开始任务。拓扑对象中操作

```java
// TridentTopology的创建方式
TridentTopology tridentTopology = new TridentTopology();
```

2.2, IRichSpout: 接口对象，通过这个接口对象接受数据

* 即使在TridentTopology中也是用相同的接口对象

```java
//IRichSpout的创建方式
继承IRichSpout，实现其方法，然后通过创建子类来创建Spout对象即可
```

2.3, TridentTopology中对Bolt和Grouping进行了封装，变成了一个个的操作方法，如each，groupBy等



2.4, 具体的使用方法：

```java
TridentTopology tridentTopology = new TridentTopology();
             tridentTopology.newStream("call-log-reader-spout", new A01_Storm01_Spout()).parallelismHint(16)
            .each( new Fields("line"), new NormalizeFunction(), new Fields("word" ))
            .groupBy( new Fields("word" ))
            .persistentAggregate( new MemoryMapState.Factory(), new Sum(), new Fields("sum" ));
StormTopology stormTopology = tridentTopology.build();
```

