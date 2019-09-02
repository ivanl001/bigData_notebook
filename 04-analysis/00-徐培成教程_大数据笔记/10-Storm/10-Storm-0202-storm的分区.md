01, storm上运行jar包：
> storm jar /mnt/hgfs/ivanl001/Desktop/everything/Storm_test.jar im.ivanl001.bigData.Storm.A03_WordCount.WordCountApp
----

## 1, shuffle

  *随机分组，具体可以参考im.ivanl001.bigData.Storm.A03_WordCount_01_shuffle_and_field包下wordcount案例，spout到WordCountSplitBolt是随机分组的*
  * 只需要在app中指定为shuffle分组方式即可
  ```java
builder.setBolt("wordCountSplitBolt", new WordCountSplitBolt(), 3).shuffleGrouping("wordCountSpout").setNumTasks(3);
  ```

## 2, field

  *根据字段分组,比较好理解，参考im.ivanl001.bigData.Storm.A03_WordCount_01_shuffle_and_field包下案例，WordCountSplitBolt到WordCountSumBolt是字段分组*
  * 只需要在app中指定为field分组方式即可  
  ```java
builder.setBolt("wordCountSumBolt", new WordCountSumBolt(), 4).fieldsGrouping("wordCountSplitBolt", new Fields("word")).setNumTasks(4);
  ```



## 3, All Group

  *类似广播，也就是每个节点(Bolt)都会收到这组消息，参考im.ivanl001.bigData.Storm.A03_WordCount_03_Allgrouping.WordCountApp*

  * 只需要在app中指定为allgroup分组方式即可
  ```java
  builder.setBolt("wordCountSplitBolt", new WordCountSplitBolt(), 3).allGrouping("wordCountSpout").setNumTasks(3);
  ```

## 4, direct group

  *只发送给指定的一个bolt*

* 01， 在app中需要设定direct分组方式
  ```java
  builder.setBolt("wordCountSplitBolt", new WordCountSplitBolt(), 3).directGrouping("wordCountSpout").setNumTasks(3);
  ```
  
* 02, 在wordCountSpout类中需要指定direct发送方式

  ```java
  public void nextTuple() {

      IMTrackerUtils.writeTo(this, "WordCountSpout----nextTuple", "master", 8880);
  
      System.out.println("---------------------------nextTuple----------------------------");
  
      String line = words.get(random.nextInt(words.size()));
      //如果用direct分组，那么这里就不能直接这样发送了
      //collector.emit(new Values(line));
  
      int directToTaskId = -999;
      //因为这里需要测试direct分组方式，所以这里先把taskId拿到，其实应该放在open中取的效率会好一些，不过测试无所谓了
      Map<Integer, String> taskToComponent = context.getTaskToComponent();
      for (Map.Entry<Integer, String> entry : taskToComponent.entrySet()) {
          if ("wordCountSplitBolt".equals(entry.getValue())){
              //这里就取第一个好了，如果有其他复杂的算法也可以在这里进行计算
              directToTaskId = entry.getKey();
              //注意：这里不能用return啊，一定要记住了！！！
              break;
          }
      }
      collector.emitDirect(directToTaskId, new Values(line));
  
      //这里为了防止发送太快，延迟一下, 已经限制数量了，这里无所谓了
      /*try {
          Thread.sleep(1000);
      } catch (InterruptedException e) {
          e.printStackTrace();
      }*/
    }
  ```

## 5, global group

  *对目标target tasked进行排序，选择最小的taskId号进行发送tuple,类似于direct,可以是特殊的direct分组。相当于特殊的direct group方式*
  * 只需要在app中指定为global group分组方式即可
  ```java
  builder.setBolt("wordCountSplitBolt", new WordCountSplitBolt(), 3).globalGrouping("wordCountSpout").setNumTasks(3);
  ```



## 6, 自定义分组(重点学习一下)

* 01，实现CustomStreamGrouping，自定义分组的类
  ```java
  package im.ivanl001.bigData.Storm.A03_WordCount_06_CustomGrouping;

  import org.apache.storm.generated.GlobalStreamId;
  import org.apache.storm.grouping.CustomStreamGrouping;
  import org.apache.storm.task.WorkerTopologyContext;
  
  import java.util.ArrayList;
  import java.util.List;
  
  /**
   * #author      : ivanl001
   * #creator     : 2018-11-20 19:31
   * #description : 自定义分组
   **/
  public class IMCustomGrouping implements CustomStreamGrouping {
  
      private List<Integer> targetTasks;
  
      public void prepare(WorkerTopologyContext context, GlobalStreamId stream, List<Integer> targetTasks) {
          this.targetTasks = targetTasks;
      }
  
      public List<Integer> chooseTasks(int taskId, List<Object> values) {
  
          //我们这里自定义直接全部分到最后一个task中
          List<Integer> choosedTasks = new ArrayList<Integer>();
          choosedTasks.add(targetTasks.get(targetTasks.size() - 1));
          return choosedTasks;
      }
  
  }
  ```
* 02, 在app中指定自定义分组方式
  ```java
  package im.ivanl001.bigData.Storm.A03_WordCount_06_CustomGrouping;

  import org.apache.storm.Config;
  import org.apache.storm.StormSubmitter;
  import org.apache.storm.generated.AlreadyAliveException;
  import org.apache.storm.generated.AuthorizationException;
  import org.apache.storm.generated.InvalidTopologyException;
  import org.apache.storm.topology.TopologyBuilder;
  
  /**
   * #author      : ivanl001
   * #creator     : 2018-11-17 09:09
   * #description : 单词统计app,自定义分组方式
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
  
          //这里定义自定义分组
          builder.setBolt("wordCountSplitBolt", new WordCountSplitBolt(), 3).customGrouping("wordCountSpout", new IMCustomGrouping()).setNumTasks(3);
  
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

---
> 下面把教程中的总计拷贝在这里以供后面学习使用
---

```java
分组策略
---------------
	1.shuffle
		随机分组.

	2.field分组
		安装指定filed的key进行hash处理，
		相同的field，一定进入到同一bolt.

		该分组容易产生数据倾斜问题，通过使用二次聚合避免此类问题。

	3.使用二次聚合避免倾斜。
		a)App入口类
		[App.java]
		/**
		 * App
		 */
		public class App {
			public static void main(String[] args) throws Exception {
				TopologyBuilder builder = new TopologyBuilder();
				//设置Spout
				builder.setSpout("wcspout", new WordCountSpout()).setNumTasks(2);
				//设置creator-Bolt
				builder.setBolt("split-bolt", new SplitBolt(),3).shuffleGrouping("wcspout").setNumTasks(3);
				//设置counter-Bolt
				builder.setBolt("counter-1", new CountBolt(),3).shuffleGrouping("split-bolt").setNumTasks(3);
				builder.setBolt("counter-2", new CountBolt(),2).fieldsGrouping("counter-1",new Fields("word")).setNumTasks(2);

				Config conf = new Config();
				conf.setNumWorkers(2);
				conf.setDebug(true);

				/**
				 * 本地模式storm
				 */
				LocalCluster cluster = new LocalCluster();
				cluster.submitTopology("wc", conf, builder.createTopology());
				//Thread.sleep(20000);
		//        StormSubmitter.submitTopology("wordcount", conf, builder.createTopology());
				//cluster.shutdown();

			}
		}


		b)聚合bolt
		[CountBolt.java]
		package com.it18zhang.stormdemo.group.shuffle;

		import com.it18zhang.stormdemo.util.Util;
		import org.apache.storm.task.OutputCollector;
		import org.apache.storm.task.TopologyContext;
		import org.apache.storm.topology.IRichBolt;
		import org.apache.storm.topology.OutputFieldsDeclarer;
		import org.apache.storm.tuple.Fields;
		import org.apache.storm.tuple.Tuple;
		import org.apache.storm.tuple.Values;

		import java.util.HashMap;
		import java.util.Map;

		/**
		 * countbolt，使用二次聚合，解决数据倾斜问题。
		 * 一次聚合和二次聚合使用field分组，完成数据的最终统计。
		 * 一次聚合和上次split工作使用
		 */
		public class CountBolt implements IRichBolt{

			private Map<String,Integer> map ;

			private TopologyContext context;
			private OutputCollector collector;

			private long lastEmitTime = 0 ;

			private long duration = 5000 ;

			public void prepare(Map stormConf, TopologyContext context, OutputCollector collector) {
				this.context = context;
				this.collector = collector;
				map = new HashMap<String, Integer>();
			}

			public void execute(Tuple tuple) {
				//提取单词
				String word = tuple.getString(0);
				Util.sendToLocalhost(this, word);
				//提取单词个数
				Integer count = tuple.getInteger(1);
				if(!map.containsKey(word)){
					map.put(word, count);
				}
				else{
					map.put(word,map.get(word) + count);
				}
				//判断是否符合清分的条件
				long nowTime = System.currentTimeMillis() ;
				if ((nowTime - lastEmitTime) > duration) {
					for (Map.Entry<String, Integer> entry : map.entrySet()) {
						//向下一环节发送数据
						collector.emit(new Values(entry.getKey(), entry.getValue()));
					}
					//清空map
					map.clear();
					lastEmitTime = nowTime ;
				}
			}

			public void cleanup() {
				for(Map.Entry<String,Integer> entry : map.entrySet()){
					System.out.println(entry.getKey() + " : " + entry.getValue());
				}
			}

			public void declareOutputFields(OutputFieldsDeclarer declarer) {
				declarer.declare(new Fields("word","count"));

			}

			public Map<String, Object> getComponentConfiguration() {
				return null;
			}
		}

	3.all分组
		使用广播分组。
		builder.setBolt("split-bolt", new SplitBolt(),2).allGrouping("wcspout").setNumTasks(2);

	4.direct(特供)
		只发送给指定的一个bolt.

		//a.通过emitDirect()方法发送元组
		//可以通过context.getTaskToComponent()方法得到所有taskId和组件名的映射
		collector.emitDirect(taskId,new Values(line));
		
		//b.指定directGrouping方式。
		builder.setBolt("split-bolt", new SplitBolt(),2).directGrouping("wcspout").setNumTasks(2);

	5.global分组
		对目标target tasked进行排序，选择最小的taskId号进行发送tuple
		类似于direct,可以是特殊的direct分组。

	6.自定义分组
		a)自定义CustomStreamGrouping类
			/**
			 * 自定义分组
			 */
			public class MyGrouping implements CustomStreamGrouping {

				//接受目标任务的id集合
				private List<Integer> targetTasks ;

				public void prepare(WorkerTopologyContext context, GlobalStreamId stream, List<Integer> targetTasks) {
					this.targetTasks = targetTasks ;
				}

				public List<Integer> chooseTasks(int taskId, List<Object> values) {
					List<Integer> subTaskIds = new ArrayList<Integer>();
					for(int i = 0 ; i <= targetTasks.size() / 2 ; i ++){
						subTaskIds.add(targetTasks.get(i));
					}
					return subTaskIds;
				}
			}

		b)设置分组策略
			public class App {
				public static void main(String[] args) throws Exception {
					TopologyBuilder builder = new TopologyBuilder();
					//设置Spout
					builder.setSpout("wcspout", new WordCountSpout()).setNumTasks(2);
					//设置creator-Bolt
					builder.setBolt("split-bolt", new SplitBolt(),4).customGrouping("wcspout",new MyGrouping()).setNumTasks(4);

					Config conf = new Config();
					conf.setNumWorkers(2);
					conf.setDebug(true);

					/**
					 * 本地模式storm
					 */
					LocalCluster cluster = new LocalCluster();
					cluster.submitTopology("wc", conf, builder.createTopology());
					System.out.println("hello world");
				}
			}



```
