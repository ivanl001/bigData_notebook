> 1, 生成数据

> 2, 启动kafka

> 3, 启动flume

## 1，生成数据
* 01， java编写代码生成电话log，具体如下:
```java
package im.ivanl001.CallLogGenSys;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.text.DecimalFormat;
import java.text.SimpleDateFormat;
import java.util.*;

/**
 * #author      : ivanl001
 * #creator     : 2018-12-14 12:13
 * #description : 日志生成程序
 **/
public class GenApp {

    public static List<String> phoneNumbers = new ArrayList<String>();
    public static Map<String,String> callers = new HashMap<String, String>();
    static{
        callers.put("158****2493", "a");
        callers.put("180****6806", "b");
        callers.put("151****9601", "c");
        callers.put("132****1119", "d");
        callers.put("150****3356", "e");
        callers.put("177****8562", "f");
        callers.put("153****5369", "g");
        callers.put("157****8050", "g");
        callers.put("156****1525", "i");
        callers.put("157****3030", "j");
        callers.put("186****1020", "k");
        callers.put("157****8446", "l");
        callers.put("133****9505", "m");
        callers.put("135****0665", "n");
        callers.put("183****9432", "o");
        callers.put("135****4983", "p");
        callers.put("183****2075", "q");
        callers.put("186****2711", "r");
        phoneNumbers.addAll(callers.keySet());
    }

    public static void main(String[] args) throws Exception {
        genCallLogs(args[0]);
    }

    private static void genCallLogs(String path) {

        FileWriter fileWriter = null;
        try{

            fileWriter = new FileWriter(new File(path), true);
            while (true) {

                //0, 随机对象
                Random random = new Random();

                //1，随机取出一个号码当作主叫
                String caller = phoneNumbers.get(random.nextInt(callers.size()));

                //2，取不是主叫号码的另外一个随机号码
                String callee = "";
                while (true) {
                    callee = phoneNumbers.get(random.nextInt(callers.size()));
                    //如果不相等就跳出去相当于选出来了
                    if (!callee.equals(caller)) {
                        break;
                    }
                }

                //3，通话时长
                int duration = random.nextInt(100);
                DecimalFormat decimalFormat = new DecimalFormat("00");
                String durationStr = decimalFormat.format(duration);

                //4,通话时间设定
                //通话时间
                int year = 2018;
                //月份(0~11)
                int month = random.nextInt(12);
                //天,范围(1~31)
                int day = random.nextInt(29) + 1;
                int hour = random.nextInt(24);
                int min = random.nextInt(60);
                int sec = random.nextInt(60);

                Calendar c = Calendar.getInstance();
                c.set(Calendar.YEAR, year);
                c.set(Calendar.MONTH, month);
                c.set(Calendar.DAY_OF_MONTH, day);
                c.set(Calendar.HOUR_OF_DAY, hour);
                c.set(Calendar.MINUTE, min);
                c.set(Calendar.SECOND, sec);
                Date date = c.getTime();

                Date now = new Date();
                if (date.compareTo(now) > 0) {
                    date = now;
                }

                SimpleDateFormat dfs = new SimpleDateFormat();
                dfs.applyPattern("yyyyMMddHHmmss");
                //通话时间
                String dateStr = dfs.format(date);

                //5, 通话log格式设定
                String log = caller + "," + callee + "," + dateStr + "," + durationStr;

                System.out.println(log);
                //注意：这里要换行
                fileWriter.write(log + "\n");
                fileWriter.flush();
                Thread.sleep(500);

            }
        }catch (Exception e){
            try {
                fileWriter.write(e.getMessage());
            } catch (IOException e1) {
                e1.printStackTrace();
            }
        }finally {
            try {
                fileWriter.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

* 02, 打成jar包，放到有flume搜集服务器上,执行以下命令开始执行java代码, 日志会生成到/data/calllogs/calllogs.txt中去
  > java -cp ./CallLog_gen_sys.jar im.ivanl001.CallLogGenSys.GenApp /data/calllogs/calllogs.txt

  ###### 这个是我自己写的一个启动命令，请忽视
  > /root/ivanl001/bin/genCalllog.sh   

## 2， 启动kafka，创建主题
* 01，kafka依赖zk，所以启动kafka之前需要先启动zk
*需要在所选服务器上分别执行如下命令，同时可以通过第二个命令查看zk状态*

  > zkServer.sh start 
  
  > zkServer.sh status
  
* 02, 启动kafka集群
*需要在所选服务器上分别执行如下命令*
  > kafka-server-start.sh /usr/local/kafka/config/server.properties &

* 03, 创建所需要的kafka主题，方便flume把收集到的数据发送过来
*4个分区，三个副本因子，也即是分成4个区，每个区保存3个副本*
  > kafka-topics.sh --create --zookeeper slave01:2181 --replication-factor 3 --partitions 4 --topic calllogs

* 04, 查看主题
  > kafka-topics.sh --list --zookeeper slave01:2181

* 05, 为了方便测试，我们这里先启动控制台消费者，消费刚才创建的calllogs主题
  > kafka-console-consumer.sh --bootstrap-server slave01:9092 --topic calllogs --from-beginning --zookeeper slave02:2181

* 06, 这里顺便给出开启生产者的方式以便调试的时候使用
  > kafka-console-producer.sh --broker-list slave01:9092 --topic calllogs

* 07，如果有需要，可以设置kafka的存活时间
*server.properties文件*
  ```java
  # The minimum age of a log file to be eligible for deletion
  # 这里是存活时间，默认是一周7*24
  log.retention.hours=168
  ```

## 3, 配置flume收集来自log文件原先数据和之后的流数据，并设置sink为kafka
* 01，编写flume的配置文件，从log文件接受，传输到kafka中去

  ```shell
  # 0, 首先指定三种组件：源,通道,槽
  agent.sources = r1
  agent.channels = c1
  agent.sinks = k1
  
  
  # 1, 定义source相关信息，绑定流入的数据， 并制定输出的方向
  agent.sources.r1.type = exec
  # 默认是最后十行,如果想要从头收集需要用-c +0, 同时-F是持续收集
  agent.sources.r1.command = tail -F -c +0 /data/calllogs/calllogs.txt
  agent.sources.r1.channels = c1
  
  
  # 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
  agent.channels.c1.type = memory
  
  
  # 3, 定义sink相关的信息
  agent.sinks.k1.channel = c1
  agent.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
  agent.sinks.k1.kafka.bootstrap.servers = slave01:9092 slave02:9092 slave03:9092
  agent.sinks.k1.kafka.topic = calllogs
  agent.sinks.k1.kafka.flumeBatchSize = 20
  agent.sinks.k1.kafka.producer.acks = 1
  ```

* 02, 使用刚才的配置文件启动flume
  > flume-ng agent -n agent -c conf -f /root/flume/conf/myConf/logs-flume-kafka.properties &

* 03，启动成功后jps可以看到多了一个Application的进程？线程？....
* 注意哈：你在哪台服务器上搜集就要在哪台服务器上启动flume哈，别搞错了

