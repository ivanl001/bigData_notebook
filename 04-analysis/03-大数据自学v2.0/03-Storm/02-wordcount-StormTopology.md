##### 1, spout

```java
package im.ivanl001.bigData.Storm.A03_WordCount_01_shuffle_and_field;

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



##### 2, 切分bolt

```java
package im.ivanl001.bigData.Storm.A03_WordCount_01_shuffle_and_field;

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



##### 3, 加和bolt

```java
package im.ivanl001.bigData.Storm.A03_WordCount_01_shuffle_and_field;

import im.ivanl001.bigData.IMUtils.IMTrackerUtils;
import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichBolt;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;

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
        this.result = new HashMap<String, Integer>();
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
        System.out.println("统计结束，开始打印结果-------------------");
        for (Map.Entry<String, Integer> entry:result.entrySet()){
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }
    }
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("IMWrodCount"));
    }
    public Map<String, Object> getComponentConfiguration() {
        return null;
    }
}
```



##### 4, app

```java
package im.ivanl001.bigData.Storm.A03_WordCount_01_shuffle_and_field;

import org.apache.storm.Config;
import org.apache.storm.StormSubmitter;
import org.apache.storm.generated.AlreadyAliveException;
import org.apache.storm.generated.AuthorizationException;
import org.apache.storm.generated.InvalidTopologyException;
import org.apache.storm.topology.TopologyBuilder;
import org.apache.storm.tuple.Fields;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-17 09:09
 * #description : 单词统计app,这里是为了方便看一下发生数据倾斜的情况。可以看到如果对spout的数据shuffleGrouping分组后，第一个bolt是ok的，但是后面如果用fieldsGrouping进行分组，很明显就会造成数据倾斜，因为次数多的单词必然会发送到同一个节点上，解决办法就是加和的bolt先进行shuffleGrouping， 计算一次后，在加和一次， 第二次用fieldsGrouping， 这样能够很好的改善数据倾斜问题，在包A03_WordCount_group02中使用这种方式
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
        builder.setBolt("wordCountSumBolt", new WordCountSumBolt(), 4).fieldsGrouping("wordCountSplitBolt", new Fields("word")).setNumTasks(4);

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



