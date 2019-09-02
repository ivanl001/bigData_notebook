> collector.ack(tuple);//成功回调
> collector.fail(tuple);//失败回调
### 下面用spout和bolt之间的消息确认机制来演示一下

#### WordCountApp,这个里面其实并没有任何消息确认机制的代码
```java
package im.ivanl001.bigData.Storm.A05_Message_fail;

import org.apache.storm.Config;
import org.apache.storm.StormSubmitter;
import org.apache.storm.generated.AlreadyAliveException;
import org.apache.storm.generated.AuthorizationException;
import org.apache.storm.generated.InvalidTopologyException;
import org.apache.storm.topology.TopologyBuilder;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-17 09:09
 * #description : 单词统计app,这里学习一下消息验证机制
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
        //builder.setBolt("wordCountSumBolt", new WordCountSumBolt(), 4).fieldsGrouping("wordCountSplitBolt", new Fields("word")).setNumTasks(4);


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

#### WordCountSpout，在这里类里面处理成功回调或者失败回调的处理办法
```java
package im.ivanl001.bigData.Storm.A05_Message_fail;

import im.ivanl001.bigData.IMUtils.IMTrackerUtils;
import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichSpout;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Values;

import java.util.*;

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

    private int i = 0;

    //消息集合，存放所有的消息，成功的需要及时删除
    private Map<Long, String> messages = new HashMap<Long, String>();
    //失败回传消息集合，需要记录重发次数
    private Map<Long, Integer> failedMsg = new HashMap<Long, Integer>();


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

        //这里限制一下发5条吧
        i += 1;
        if (i > 5) {
            return;
        }

        System.out.println("---------------------------nextTuple----------------------------");

        String line = words.get(random.nextInt(words.size()));

        //这里是用时间戳作为消息id，后面接收到回调的时候方便根据消息id获取内容
        Long timestamp = System.currentTimeMillis();
        //发送之前先存到message中，方便成功回传后删除， 失败回传后获取重新发送
        messages.put(timestamp, line);

        collector.emit(new Values(line), timestamp);

        IMTrackerUtils.writeTo(this, "WordCountSpout----nextTuple----" + line + "-----" + timestamp , "master", 8880);

        //这里为了防止发送太快，延迟一下, 已经限制数量了，这里无所谓了
        /*try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }*/
    }

    public void ack(Object timstamp) {
        Long theTimestamp = (Long) timstamp;


        Integer retrytimes = failedMsg.get(theTimestamp);
        retrytimes = (retrytimes == null) ? 0 : retrytimes;
        IMTrackerUtils.writeTo(this, "WordCountSpout----ack-----" + theTimestamp + "----次数---" + retrytimes, "master", 8880);

        failedMsg.remove(theTimestamp);
        messages.remove(theTimestamp);

    }

    public void fail(Object timstamp) {
        //如果出错了，先从错误消息中取出来看看是否存在，是否已经重发3次
        Long theTimestamp = (Long) timstamp;
        Integer retrytimes = failedMsg.get(theTimestamp);
        retrytimes = (retrytimes == null) ? 0 : retrytimes;
        IMTrackerUtils.writeTo(this, "WordCoun 
        }

    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("line"));
    }

    public Map<String, Object> getComponentConfiguration() {
        return null;
    }
}
```

#### WordCountSplitBolt, 这里进行成功回调发送或者失败回调发送
```java
package im.ivanl001.bigData.Storm.A05_Message_fail;

import im.ivanl001.bigData.IMUtils.IMTrackerUtils;
import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichBolt;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;

import java.util.Map;
import java.util.Random;

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

            if (new Random().nextBoolean()) {
                //这里是消费成功，进行确认，发送的节点，这里发送的节点也就是spout，会收到回调通知
                collector.ack(tuple);
            }else{
                //随机让失败一些，方便我们处理，如果失败的话， 需要重新发送，并需要设定重新发送的最高次数，不能一直重复一直发送
                collector.fail(tuple);
            }

            /*String[] words = line.split(" ");
            for (String word : words) {
                collector.emit(new Values(word, 1));
            }*/
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