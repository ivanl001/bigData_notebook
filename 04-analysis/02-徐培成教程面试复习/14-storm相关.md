0，storm属于无状态的实时流计算，能保证at lease once,但是吞吐量应该比flink要小，单独部署，不需要依赖yarn等，适合小的实时程序。



1,  数据分组方式方式和mr的分区类似，storm的wordcount案例中是用fieldsGrouping的，但是这样会造成数据倾斜，解决办法也和mr类似，

* 用shuffleGrouping，也就是直接混洗后计算出数量，然后再用fieldsGrouping把相同的单词分区到一起，用两次聚合避免倾斜



2, storm可以和kafka集成，消费kafka主题，也可以和hbase集成，直接写入到hbase中

和kafka集成，直接用设置spout为new KafkaSpout(spoutConfig)即可，具体看代码或者storm官方文档



3，storm的消息确认

collector.ack(tuple);//成功回调
collector.fail(tuple);//失败回调



在处理成功的那个bolt中进行消息回调，

然后在接受数据的spout中会接受到回调，如果失败，重新发送消息即可，也可以自定义重发次数。



具体的spout类如下：

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

