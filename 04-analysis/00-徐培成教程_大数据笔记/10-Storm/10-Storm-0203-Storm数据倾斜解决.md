*其实数据倾斜的问题很好解决，还是拿wordcount来举例，第一次算和的时候采用shuffle随机分组，然后第二次再采用一次field分组方式，最后一个bolt来收集计算的数据即可，具体代码参考如下：*

```
一共有六个类：
WordCountApp类：    主app入口
WordCountSpout类：  输入
WordCountSplitBolt：语句切割bolt节点，
WordCountSumBolt：  这个计算总和的类中，需要注意的是解决数据倾斜需要用到两次这个类，一次用shuffle随机分组，一次用field分组进行单个单词的汇总
WordCountResultBolt: 在这里进行最后数据的搜集
```

####  WordCountApp类

```java
package im.ivanl001.bigData.Storm.A03_WordCount_02_secondary;

import org.apache.storm.Config;
import org.apache.storm.LocalCluster;
import org.apache.storm.StormSubmitter;
import org.apache.storm.generated.AlreadyAliveException;
import org.apache.storm.generated.AuthorizationException;
import org.apache.storm.generated.InvalidTopologyException;
import org.apache.storm.topology.TopologyBuilder;
import org.apache.storm.tuple.Fields;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-17 09:09
 * #description : 单词统计app， 解决数据倾斜问题:这里使用的是二次聚合的方法，比较不同的是：WordCountSumBolt中的值需要传递出去，给第二次的聚合使用
 *
 * ??但是应该好是有一个问题的：在sumBolt中把结果发送出去后就把map清空了， 那结果怎么搜集？(意思是WordCountSumBolt要使用两次，第一次随机分组，第二次才按字段分组，这样可以规避数据倾斜，但是WordCountSumBolt既然要用两次，那肯定是要把数据发送出去，第二次再使用的时候才能接受对吧，但是第二次使用的时候代码中也有发送出去的代码，在发送出去的时候刚好也把map清空了， 那结果map的数据怎么能搜集到呢？)
 *
 * 后面想到一个办法，就是重新再写一个bolt最后接受数据就好了，笨
 *
 **/
public class WordCountApp {

    public static void main(String[] args) throws InterruptedException, InvalidTopologyException, AuthorizationException, AlreadyAliveException {

        Config config = new Config();
        config.setDebug(false);
        //这个是设置开启几个worker
        config.setNumWorkers(2);

        TopologyBuilder builder = new TopologyBuilder();

        //parallelism_hint是设置并发的数量，比如下面spout设置成2,那么会开两个线程，跑3个任务，这个时候一个线程就必须同时跑两个任务，这其实不太好，所以并发数需要尽量大于任务数
        builder.setSpout("wordCountSpout", new WordCountSpout(), 2).setNumTasks(2);

        builder.setBolt("wordCountSplitBolt", new WordCountSplitBolt(), 3).shuffleGrouping("wordCountSpout").setNumTasks(3);
        //计算总数的时候为了解决数据倾斜的问题，这里需要进行两次计算总数，第一次混洗分组，第二次根据字段分组
        builder.setBolt("wordCountSumBolt_shuffle", new WordCountSumBolt(), 4).shuffleGrouping("wordCountSplitBolt").setNumTasks(4);
        builder.setBolt("wordCountSumBolt_field", new WordCountSumBolt(), 1).fieldsGrouping("wordCountSumBolt_shuffle", new Fields("word")).setNumTasks(1);
        builder.setBolt("wordCountResultBolt", new WordCountResultBolt(), 1).fieldsGrouping("wordCountSumBolt_field", new Fields("word")).setNumTasks(1);//最后搜集

        //这里用本地模式来测试分区模式，消息还是发送到master上的nc好了，懒得改了，一样看
        /*LocalCluster localCluster = new LocalCluster();
        localCluster.submitTopology("localWordCountCluster", config, builder.createTopology());
        Thread.sleep(10000);
        localCluster.shutdown();*/

        //集群提交
        StormSubmitter.submitTopology("wordCountCluster", config, builder.createTopology());
    }
}
```

####  WordCountSpout类
```java
package im.ivanl001.bigData.Storm.A03_WordCount_02_secondary;

import im.ivanl001.bigData.IMUtils.IMTrackerUtils;
import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichSpout;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Values;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Random;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-17 09:09
 * #description : 单词统计的源
 **/
public class WordCountSpout implements IRichSpout {

    //在这里定义一些输入，方便后面的Bolt进行计算
    //private TopologyContext context;
    private SpoutOutputCollector collector;
    private List<String> words;
    private Random random;

    public void open(Map map, TopologyContext topologyContext, SpoutOutputCollector spoutOutputCollector) {

        IMTrackerUtils.writeTo(this, "WordCountSpout----open", "master", 8880);

        //this.context = topologyContext;
        this.collector = spoutOutputCollector;
        words = new ArrayList<String>();

        words.add("ivanl001 is the king of world!");
        words.add("ivanl002 is the king of world!");
        words.add("ivanl003 is the king of world!");
        words.add("ivanl004 is the king of world!");
        words.add("ivanl005 is the king of world!");
        words.add("ivanl006 is the king of world!");

        random = new Random();
    }

    public void close() {

    }

    public void activate() {

    }

    public void deactivate() {

    }

    public void nextTuple() {

        IMTrackerUtils.writeTo(this, "WordCountSpout----nextTuple", "master", 8880);

        System.out.println("---------------------------nextTuple----------------------------");

        String line = words.get(random.nextInt(words.size()));
        collector.emit(new Values(line));

        //这里为了防止发送太快，延迟一下, 已经限制数量了，这里无所谓了
        /*try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }*/
    }

    public void ack(Object o) {

    }

    public void fail(Object o) {

    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("line"));
    }

    public Map<String, Object> getComponentConfiguration() {
        return null;
    }
}
```

#### WordCountSplitBolt类
```java
package im.ivanl001.bigData.Storm.A03_WordCount_02_secondary;

import im.ivanl001.bigData.IMUtils.IMTrackerUtils;
import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichBolt;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

import java.util.Map;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-17 09:21
 * #description : 分裂语句节点
 **/
public class WordCountSplitBolt implements IRichBolt {

    //private TopologyContext context;
    private OutputCollector collector;
    private int num = 0;

    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {

        IMTrackerUtils.writeTo(this, "WordCountSplitBolt----prepare", "master", 8881);

        //this.context = topologyContext;
        this.collector = outputCollector;
    }


    //在这里执行计算操作
    public void execute(Tuple tuple) {

        //这里只让发送五行
        if (num < 5) {

            //这里可以通过字段名获取，也可以通过索引获取
            String line = tuple.getString(0);
            IMTrackerUtils.writeTo(this, "WordCountSplitBolt----execute" + ":::" + line, "master", 8881);

            String[] words = line.split(" ");
            for (String word : words) {
                collector.emit(new Values(word, 1));
            }

            num += 1;
        }
    }

    public void cleanup() {

    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("word", "count"));
    }

    public Map<String, Object> getComponentConfiguration() {
        return null;
    }
}
```

#### WordCountSumBolt
```java
package im.ivanl001.bigData.Storm.A03_WordCount_02_secondary;

import im.ivanl001.bigData.IMUtils.IMTrackerUtils;
import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichBolt;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

import java.util.Collections;
import java.util.HashMap;
import java.util.Map;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-17 09:26
 * #description : 计算总数Bolt节点
 **/
public class WordCountSumBolt implements IRichBolt {

    //private TopologyContext context;
    private OutputCollector collector;

    Map<String, Integer> result;

    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {

        IMTrackerUtils.writeTo(this, "WordCountSumBolt----prepare", "master", 8882);

        //this.context = topologyContext;
        this.collector = outputCollector;
        result = new HashMap<String, Integer>();
        //转成线程安全的，方便后面另起线程清空操作等
        result = Collections.synchronizedMap(result);

        //在这里开启一个线程循环发送数据，因为如果只是在execute中发送的时候，有可能前面不发送数据，execute就不被触发，有一部分数据就发送不出去了
        Thread thread = new Thread(){
            @Override
            public void run() {
                //这里需要一直循环下去，一旦线程歇菜就完蛋了哈
                while (true) {
                    emitData();
                }
            }
        };
        //这个进行会循环，如果直接start只有一次，不是特别懂
        thread.setDaemon(true);
        thread.start();
    }

    //在这个线程中进行清理数据发送给下以阶段
    public void emitData(){

        //其实这里还是有一点不明白的，这里用result加锁了， 那么result是不是线程安全的是不是就无所谓了？？？
        synchronized (result) {
            //这里的result集合是HashMap， 是非线程安全的，所以为了线程问题，需要转换成线程安全的
            for (Map.Entry<String, Integer> entry : result.entrySet()) {
                collector.emit(new Values(entry.getKey(), entry.getValue()));
                System.out.println("发送数据啦，数据个数是：" + result.size());
            }
            result.clear();
        }

        try {
            //这里一定要休眠一下，要不然会一直调用，很浪费
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    //在这里计算单词总数
    public void execute(Tuple tuple) {

        String word = tuple.getStringByField("word");
        Integer count = tuple.getIntegerByField("count");

        IMTrackerUtils.writeTo(this, "WordCountSumBolt----execute" + ":::" + word, "master", 8882);

        if (result.containsKey(word)) {
            //已经包含了这个单词,那么取出原先的数量，加上本次数量
            result.put(word, result.get(word) + count);
        } else {
            result.put(word, count);
        }
        //collector.ack(tuple);

    }

    public void cleanup() {

        //这里再统计就没有意义了，在WordCountResultBolt中进行搜集统计数据
        /*System.out.println("统计结束，开始打印结果-------------------");
        for (Map.Entry<String, Integer> entry:result.entrySet()){
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }*/
    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("word", "count"));
    }

    public Map<String, Object> getComponentConfiguration() {
        return null;
    }
}
```

#### WordCountResultBolt类
```java
package im.ivanl001.bigData.Storm.A03_WordCount_02_secondary;

import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichBolt;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Tuple;

import java.util.HashMap;
import java.util.Map;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-20 10:21
 * #description : 这里是为了搜集到数据，因为之前的bolt都把数据发送出去了，所以必须要找个bolt接受一下然后搜集起来对吧
 **/
public class WordCountResultBolt implements IRichBolt {

    Map<String, Integer> result;

    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
        result = new HashMap<String, Integer>();
    }

    public void execute(Tuple tuple) {
        String word = tuple.getStringByField("word");
        Integer count = tuple.getIntegerByField("count");
        result.put(word, count);
    }


    public void cleanup() {
        System.out.println("统计结束，开始打印结果-------------------");
        for (Map.Entry<String, Integer> entry:result.entrySet()){
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }
    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {

    }

    public Map<String, Object> getComponentConfiguration() {
        return null;
    }
}
```