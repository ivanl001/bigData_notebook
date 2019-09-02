[http://www.tianshouzhi.com/api/tutorials/storm/56](http://www.tianshouzhi.com/api/tutorials/storm/56)



# 4.0 Trident简介

`Trident`是Storm中最为核心的概念，在做Strom开发的过程中，绝大部分情况下我们都会使用Trident，而不是使用传统的Spout、Bolt。Trident是Storm原语的高级封装，学会Trident之后，将会使得我们Storm开发变得非常简单。

**一、什么是Storm Trident ？**

简而言之：Trident是编写Storm Topology的一套高级框架，是对传统Spout、Bolt的高级封装。在学习Trident之前，我们都是都Spout、Bolt的相关API来编写一个Topology，在学习了Trident之后，我们会使用Trident API来编写Topology。

可以将StormTopology与`TridentTopology`的关系，类比为JDBC与ORM框架(mybatis、hibernate)之间的关系，后者是前者的高级封装，功能相同，但是可以极大的减少我们的开发的工作量。

当然，就像我们学习JDBC与ORM框架一样，JDBC可能很容易理解，但是学习ORM框架，可能就相对复杂一点。甚至学习ORM框架的时间可能要比学习JDBC的时间要更长，但是一旦我们学会了ORM框架，可能就再也不想去使用JDBC了，因为ORM框架可以帮助我们更高效的进行开发。

学习Trident也一样，可能我们学习理解许多新的概念，但是学会了会极大的提高我们的开发效率。

Storm原语中，最重要的就是Spout、Bolt、Grouping等概念。

Trident对于Storm原语的抽象主要也就是针对这些基本概念的抽象。主要体现在：Trident Spout,Operation、State 。

**Trident Spout**是针对Storm原语中的Spout进行的抽象

**Operation**是针对Bolt、Grouping等概念的抽象

**State**是新提出的概念，实际上就是数据持久化的接口。



通常情况下，新的概念意味着要使用新的API。但是归根结底，还是底层还是通过storm原语来实现。在Trident中，我们使用TridentTopology表示一个拓扑，而在Storm原语中，我们使用StormTopology来表示一个拓扑。TridentTopology最终会被转换成StormTopology。在接下来的内容，我们将首先介绍TridentTopology的构建过程，以及TridentTopology如何转化为StormTopology。

**二、TridentTopology与StormToplogy**

**1、API区别：**

在Trident中，有着新的一套构建Topology的API，我们先通过从代码层面上对比来进行分析：

StormTopology：由传统的Spout和Bolt的API编写的Topology，最终是通过`TopologyBuilder`对象来创建的，返回的结果就是StormTopology对象。

```java
StormTopologytopology= topologyBuilder.createTopology();
```

TridentTopology：由Trident的API编写的Topology，因为在Trident的API中，使用TridentTopology来表示一个Topology，是直接new出来的。

```java
TridentTopology tridentTopology = new TridentTopology();
```

单独提出这两个概念，是为了以后的区分。以后我们提到StormTopology时，表示的就是以Spout、Bolt等这些API创建的Topology，而提到TridentTopology表示的就是以Trident的API创建的Topology。

**2、创建Topology的区别**

我们以单词计数案例WordCountApp(超链接)进行对比

StormTopology

```java
TopologyBuilder builder = new TopologyBuilder();  builder.setSpout("word-reader" , new WordReader(),4);  builder.setBolt("word-normalizer" , new WordNormalizer(),3).shuffleGrouping("word-reader" );  builder.setBolt("word-counter" , new WordCounter(),1).fieldsGrouping("word-normalizer" , new Fields("word"));  StormTopology topology = builder .createTopology();
```

TridentTopology：

```java
TridentTopology tridentTopology = new TridentTopology();             tridentTopology.newStream("word-reader-stream" , new WordReader()).parallelismHint(16)            .each( new Fields("line" ), new NormalizeFunction(), new Fields("word" ))            .groupBy( new Fields("word" ))            .persistentAggregate( new MemoryMapState.Factory(), new Sum(), new Fields("sum" ));StormTopology stormTopology = tridentTopology.build();
```

**对比：**

在StormTopology中，我们都是通过TopologyBuilder的setSpout、setBolt的方式来创建Topology，然后通过Grouping策略指定Bolt的数据来源和分组策略。

在TridentTopology中，我们使用TridentTopology来创建Topology，整个创建过程中，都是流式编程风格的。要注意的是，在Trident中，我们依然使用了WordReader这个Spout，但是并没有使用Bolt，而是使用了类似于each、persistentAggregate这样方法，来取代Bolt的功能。关于这些方法的作用再之后会详细介绍，目前只要知道Bolt的作用被一些方法取代了即可。

**三、TridentTopology与StormToplogy的联系**

二者的联系主要是：TridentTopology最终会被编译成StormTopology。请再次查看上述构建构建TridentTopology的最后一句代码。

```java
StormTopology stormTopology = tridentTopology.build();
```

这句代码的的返回结果还是StormTopology对象，这实际上意味着，TridentTopology最终还是会被编译成StandardTopology。这很容易理解，就像ORM框架与JDBC一样，ORM框架只是一层封装，最终还是要通过JDBC操作数据库。而TridentTopology是高级封装，但是最终还是要通过编译StormTopology来运行。

**注意：**这一点是非常重要的。上面我们已经提到，在Trident中，依然要指定Spout，但是用了一系列其他的方法如each、persistentAggregate等（当然不止这些），代替了Bolt的功能。那么这里又提到，TridentTopology会被编译成StormTopology，实际上就意味着Storm最终会将这些方法转换为一个或多个Bolt。我们要了解Trident是如何工作的，就必须要了解，这些方法的最终是如何被转换为Bolt的。最简单的查看方式，就是查看编译后的StormTopology的getSpout，getBolt方法来看。



 在后面我们会详细介绍，TridentTopology是如何转换为StormTopology的。目前，我们只需要知道TridentTopology最终是会转换为StormTopology即可。

事实上，在Trident框架会将调用的所有方法转换为一个个Node。Node类型如下：

newStream方法中参数Spout转换为SpoutNode，将调用的each方法，persistentAggregate等方法，转换为一个个ProcesserNode，而groupBy等操作，转换为一个PartitionNode，最终组成一个对象图，最后根据这个对象图，来将TridentTopology转换为StormTopology。

- 当然，光说不练假把式，我们通过分析源码进行简单说明。
- 首先说明Spout转换为SpoutNode对象
- 其次说明each、persistentAggregate转换为ProcesserNode
- 最后说明如何根据SpoutNode和ProcesserNode将TridentTopology转换为StormTopology



1、Spout转换为SpoutNode**

首先，我们看一下TridentTopology对象的newStream方法：

![Image.png](01-Storm编程基础03-两种方式比较.assets/1454257631745092261.png)

可以看到，可以接受五种类型的Spout，以下是这五个方法的实现：

![Image.png](01-Storm编程基础03-两种方式比较.assets/1454257657691024653.png)

我们可以看到，这五种方法，最终调用的实际上只有两个，并且在这两个方法中，最终都将Spout转换为了`SpoutNode`对象。



**2、each、persistentAggregate转换为ProcessorNode**

我们可以查看Stream对象的源码， 找到each、persistentAggregate两个方法内容

![Image.png](01-Storm编程基础03-两种方式比较.assets/1454257688359098042.png)

上图显示了这两个方法，这种都被转换为一个`ProcessorNode`，最终添加到Topology中。

**3、最后我们看一下，TridentTopology.build()的实现**

由于源码内容比较多，我们只分析感兴趣的地方，其中红色加深的地方，是目前最为关注的：

```java
public StormTopology build() {
...
TridentTopologyBuilder builder = new TridentTopologyBuilder();
       
        Map<Node, String> spoutIds = genSpoutIds( spoutNodes);
        Map<Group, String> boltIds = genBoltIds( mergedGroups);
        // SpoutNode维护了Spout类型，根据类型转换为对应的Spout
        for(SpoutNode sn : spoutNodes ) {
            Integer parallelism = parallelisms.get(grouper .nodeGroup(sn));
            if(sn .type == SpoutNode.SpoutType.DRPC) {
                builder.setBatchPerTupleSpout(spoutIds .get(sn), sn.streamId ,
                        (IRichSpout) sn. spout, parallelism , batchGroupMap.get(sn ));
            } else {
                ITridentSpout s;
                if(sn .spout instanceof IBatchSpout) {
                    s = new BatchSpoutExecutor((IBatchSpout)sn .spout );
                } else if(sn .spout instanceof ITridentSpout) {
                    s = (ITridentSpout) sn. spout;
                } else {
                    throw new RuntimeException("Regular rich spouts not supported yet... try wrapping in a RichSpoutBatchExecutor");
                    // TODO: handle regular rich spout without batches (need lots of updates to support this throughout)
                }
                builder.setSpout(spoutIds .get(sn), sn.streamId, sn. txId, s, parallelism , batchGroupMap .get(sn));
            }
        }
       
        for(Group g : mergedGroups ) {
            if(!isSpoutGroup( g)) {
                Integer p = parallelisms.get(g );
                Map<String, String> streamToGroup = getOutputStreamBatchGroups(g, batchGroupMap);
                //将调用each、processAggregate方法后的ProcessorNode，转换为Bolt
                BoltDeclarer d = builder.setBolt(boltIds .get(g), new SubtopologyBolt(graph, g .nodes , batchGroupMap ), p,
                        committerBatches(g, batchGroupMap), streamToGroup);
                Collection<PartitionNode> inputs = uniquedSubscriptions(externalGroupInputs(g ));             //根据调用GroupBy等方法转换成的PartitionNode进行Grouping策略
                for(PartitionNode n : inputs ) {
                    Node parent = TridentUtils.getParent( graph, n );
                    String componentId;
                    if(parent instanceof SpoutNode) {
                        componentId = spoutIds .get(parent);
                    } else {
                        componentId = boltIds.get(grouper .nodeGroup(parent));
                    }
                    d.grouping( new GlobalStreamId(componentId , n.streamId ), n.thriftGrouping);
                }
            }
        }
       
        return builder .buildTopology();
}
```



上一篇：[2.9 storm监控--Zookeeper](http://www.tianshouzhi.com/api/tutorials/storm/22)下一篇：[4.0 Trident快速入门](http://www.tianshouzhi.com/api/tutorials/storm/57)