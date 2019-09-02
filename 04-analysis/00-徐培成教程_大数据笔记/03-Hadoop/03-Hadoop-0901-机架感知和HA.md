fault tolerance   容错
fail over         容灾

## 1，机架感知
*我们这里使用jar包的方式，也就是实现DNSToSwitchMapping，重写resolve方法*

### 1.1，首先定义一个机架感知类，实现DNSToSwitchMapping

```java
package im.ivanl001.bigData.Hadoop.A16_RackAwareness;

import org.apache.hadoop.net.DNSToSwitchMapping;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * #author      : ivanl001
 * #creator     : 2018-10-30 18:03
 * #description : 机架感知
 **/
public class IMRackAwareness implements DNSToSwitchMapping{


    /*
     * 简单解释一下：
     * /rack1/a
     * /rack1/b
     * /rack2/c
     * /rack3/d
     * 上面表示a,b表示在同一机架, c或者d不和其他任何机器在同一机架
     * 同样的，也可以用三层表示是否在同一更深的层次
     * /level1/rack1/a
     * /level2/rack1/b
     * /level1/rack1/a
     */

    //通过实现这个方法来定义机架感知的规则
    public List<String> resolve(List<String> names) {

        List<String> list = new ArrayList<String>();

        try {

            FileWriter fileWriter = new FileWriter("/usr/local/hadoop-2.7.5/logs/rackAwareness.log", true);
            for (String str : names) {

                fileWriter.write(str + "\n");

                //
                String end = str.substring(str.length()-1, str.length());

                //在实际上，这里会有一遍hosts，然后还有有一遍ip，所以我们都解析一下，按照我的理解，如果你解析其中的一种应该也是没问题的，不过并不确定
                //主机名
                if (str.startsWith("slave")) {
                    //在DNSToSwitchMapping的注释中可以看到，names其实是需要解析的hosts，也就是主机名
                    //在我的系统中，其实也就是slave01， slave02， slave03, slave04
                    //为了验证可用性，我这里把01，04解析到同一机架，02，03解析到同一机架
                    //String end = str.substring(str.length()-1, str.length());

                    if ("slave01".equals(str) || "slave04".equals(str)) {
                        list.add("/rack1/" + end);
                        fileWriter.write("hostname----：/rack1/" + end + "\n");
                    } else if ("slave02".equals(str) || "slave03".equals(str)) {
                        list.add("/rack2/" + end);
                        fileWriter.write("hostname----：/rack2/" + end + "\n");
                    }
                } else if (str.startsWith("192")) {
                    //取出最后一个点后面的
                    String ip_end = str.substring(str.lastIndexOf(".") + 1);
                    if ("121".equals(ip_end) || "124".equals(ip_end)) {
                        list.add("/rack1/" + end);
                        fileWriter.write("IP----：/rack1/" + end + "\n");
                    } else if ("122".equals(ip_end) || "123".equals(ip_end)) {
                        list.add("/rack2/" + end);
                        fileWriter.write("IP----：/rack2/" + end + "\n");
                    }
                }
            }

            fileWriter.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
        return list;
    }

    public void reloadCachedMappings() {
    }

    public void reloadCachedMappings(List<String> names) {
    }
}
```

### 1.2, 把架包放到hadoop的namenode节点上的指定包中
`/usr/local/hadoop-2.7.5/share/hadoop/common/lib`

### 1.3, 在配置文件中添加配置信息, 
`在core-site.xml中添加如下的配置信息，应该在namenode节点上添加即可(有点需要注意的是：低版本的hadoop这个属性name可能会有所不同)：`
```xml
<!-- 指定机架感知的是用那个类 -->
<property>
  <name>net.topology.node.switch.mapping.impl</name>
  <value>im.ivanl001.bigData.Hadoop.A16_RackAwareness.IMRackAwareness</value>
</property>
```

### 1.4，重启一下namenode节点
`重启之后应该能够看到我们在代码中定义在"/usr/local/hadoop-2.7.5/logs/rackAwareness.log"的log文件`

> 有一个问题，就是按照正常的步骤进行机架感知，总是会报错，报错如下：暂时不知道为什么，所以机架感知先放着，有机会问一下
```
2018-10-31 12:37:23,129 WARN org.apache.hadoop.ipc.Server: IPC Server handler 2 on 8020, call org.apache.hadoop.hdfs.protocol.ClientProtocol.addBlock from 192.168.217.120:47430 Call#5 Retry#0
java.lang.IndexOutOfBoundsException: Index: 0, Size: 0
        at java.util.ArrayList.rangeCheck(ArrayList.java:653)
        at java.util.ArrayList.get(ArrayList.java:429)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getClientNode(FSNamesystem.java:3203)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getNewBlockTargets(FSNamesystem.java:3123)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getAdditionalBlock(FSNamesystem.java:3051)
        at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.addBlock(NameNodeRpcServer.java:725)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.addBlock(ClientNamenodeProtocolServerSideTranslatorPB.java:493)
        at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:616)
        at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:982)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2217)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2213)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:422)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1754)
        at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2213)
```

## 2，使用QJM实现高可用

*之前因为其中一个配置文件的serverId没有改成我自己定义的那个名字，也就是imcluster,所以报错没有启动好，后来发现后ok了*

官方文档：

### 01，QJM的介绍

* 1， QJM全称是：Quorum Journal Manager
* 2， 使用QJM后，编辑日志就不会存放在namenode上了，而是放在了QJN(可以理解为QJM的数据节点)上(为了保证高可用，需要最少配置三台QJN)，不再需要配置secondarynamenode，然后namenode和QJM交互，QJM中也有两个节点，其中一个状态是active，另外一个状态是standby
* 3，QJM中的active和standby的两个名称节点需要保持和QJN的通讯，保持信息为最新，因为数据编辑日志现在是存在QJN上了。然后datanode也需要同时和QJM的那两个名称节点关联，以保证能快速容灾
* 
* In order to provide a fast failover, it is also necessary that the Standby node have up-to-date information regarding the location of blocks in the cluster. In order to achieve this, the DataNodes are configured with the location of both NameNodes, and send block location information and heartbeats to both.
* 
* It is vital for the correct operation of an HA cluster that only one of the NameNodes be Active at a time. Otherwise, the namespace state would quickly diverge between the two, risking data loss or other incorrect results. In order to ensure this property and prevent the so-called “split-brain scenario,” the JournalNodes will only ever allow a single NameNode to be a writer at a time. During a failover, the NameNode which is to become active will simply take over the role of writing to the JournalNodes, which will effectively prevent the other NameNode from continuing in the Active state, allowing the new Active to safely proceed with failover.

### 02，QJM的完整配置

#### 2.1, hdfs-site.xml中新增配置

```xml
<!-- 设置QJM服务名字  -->
<property>
  <name>dfs.nameservices</name>
  <value>imcluster</value>
</property>

<!-- 设置QJM服务中名称节点的名字 -->
<property>
  <name>dfs.ha.namenodes.imcluster</name>
  <value>nn1,nn2</value>
</property>

<!-- 两个QJM的名称节点的rpc地址配置 -->
<property>
  <name>dfs.namenode.rpc-address.imcluster.nn1</name>
  <value>master:8020</value>
</property>
<property>
  <name>dfs.namenode.rpc-address.imcluster.nn2</name>
  <value>master01:8020</value>
</property>

<!-- 两个QJM的名称节点的webui地址配置 -->
<property>
  <name>dfs.namenode.http-address.imcluster.nn1</name>
  <value>master:50070</value>
</property>
<property>
  <name>dfs.namenode.http-address.imcluster.nn2</name>
  <value>master01:50070</value>
</property>

<!-- active的QJM名称节点写入的数据节点，QJNs, 我这里配置slave01，slave02, slave03三个节点  -->
<property>
  <name>dfs.namenode.shared.edits.dir</name>
  <value>qjournal://slave01:8485;slave02:8485;slave03:8485/imcluster</value>
</property>
<!-- QJN存放的具体路径 -->
<property>
  <name>dfs.journalnode.edits.dir</name>
  <value>/data/QJN/data</value>
</property>


<!-- 这个是固定的，容灾的时候使用的类，直接原文配置即可  -->
<property>
  <name>dfs.client.failover.proxy.provider.imcluster</name>
  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>
<!-- SSH to the Active NameNode and kill the process，也是容灾配置，原文配置 -->
<property>
  <name>dfs.ha.fencing.methods</name>
  <value>sshfence</value>
</property>

<!-- 这里教程里这样写，但是我按照上面一行的配置也是可以的，所以暂且不究 -->
 <!-- <property>
	<name>dfs.ha.fencing.methods</name>
	<value>
			sshfence
			shell(/bin/true)
	</value>
</property> -->

<!-- 这里需要找到rsa方便可以杀死先前的active节点 -->
<property>
  <name>dfs.ha.fencing.ssh.private-key-files</name>
  <value>/root/.ssh/id_rsa</value>
</property>
```

#### 2.2, core-site.xml中新增配置
```xml
<!-- 那么默认的文件系统就不能直接写原来的namenode节点了，而需要写我们QJM服务节点  -->
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://imcluster</value>
</property>
```

### 03, 部署细节

#### 3.0, 先关闭所有的节点 stop-all.sh

#### 3.1，在QJN分别启动jn，三个节点都需要启动
`hadoop-daemon.sh start journalnode`

#### 3.2，新增节点需要同步数据信息，因为我本身是直接clone的，所以这里就不需要了

#### 3.2，把另外一个新增的节点上执行如下命令，准备该节点
* 注意：先需要把另外一个namenode节点开启
* `hadoop-daemon.sh start namenode`

`hdfs namenode -bootstrapStandby`

#### 3.3, 执行一下命令，从namenode节点把数据传输到QJN上
`hdfs namenode -initializeSharedEdits`

#### 3.4, 启动其余的其他所有节点
* 启动另外一个名称节点
`hadoop-daemon.sh start namenode`
* 启动所有的数据节点
`hadoop-daemons.sh start datanode`



```text

部署细节
----------------
	1.在jn节点分别启动jn进程
		$>hadoop-daemon.sh start journalnode

	2.启动jn之后，在两个NN之间进行disk元数据同步
		a)如果是全新集群，先format文件系统,只需要在一个nn上执行。
			[s201]
			$>hadoop namenode -format
		
		b)如果将非HA集群转换成HA集群，复制原NN的metadata到另一个nn.
			1.步骤一
				[s201]
				$>scp -r /home/centos/hadoop/dfs centos@s206:/home/centos/hadoop/

			2.步骤二
				在新的nn(未格式化的nn)上运行一下命令，实现待命状态引导。
				[s206]
				$>hdfs namenode -bootstrapStandby		//需要s201为启动状态,提示是否格式化,选择N.
				
			3)在一个NN上执行以下命令，完成edit日志到jn节点的传输。
				$>hdfs namenode -initializeSharedEdits
				#查看s202,s203是否有edit数据.

			4)启动所有节点.
				[s201]
				$>hadoop-daemon.sh start namenode		//启动名称节点
				$>hadoop-daemons.sh start datanode		//启动所有数据节点

				[s206]
				$>hadoop-daemon.sh start namenode		//启动名称节点
```

### 04，管理命令
`hdfs haadmin`
* 这里相当于把nn2的激活状态改成standby，然后nn1改成激活状态
`hdfs haadmin -failover nn2 nn1`

