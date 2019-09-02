> 10, hive的集成使用

> 11, Yarn设置内存限制

## 10, Hive

### 011, 安装
*Hive安装，初始化等就不再赘述，可以看前面等hive章节内容*

### 02，创建外部表链接hbase上

* 01, 链接表
  > create external table calllogs_hive_hbase(id string, caller string, time string, callee string, duration string) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key, f1:caller, f1:time, f1:callee, f1:duration") TBLPROPERTIES ("hbase.table.name" = "ns3:calllogs");

* 02, 链接过后就可以按照正常sql进行查询，id就是对应rowkey
  > select * from calllogs_hive_hbase;
  > select * from calllogs_hive_hbase where caller = '15732648446'
  > select * from calllogs_hive_hbase where id like '%13269361119%'

* 03, 类似下面的会转成MR
  * 排序的
    > select * from calllogs_hive_hbase where id like '%13269361119%' order by time desc;
  
  * 聚合的
    > select count(*) from calllogs_hive_hbase where id like '%13269361119%';

* 04, 如果想在代码中访问hive，需要启动hiveserver2, jdbc访问
    > hive/bin/hiveserver2 & 

### 03, web程序中代码

*注意：⚠️：hive和hbase的maven架包极大可能会发生冲突，表面上web程序启动正常，但是在实际上是创建连接或者其他时候会发生错误。所以一定要按照不同的错误进行分析，我这里给出我解决后的可以正常使用的maven依赖，不过在不同的情况下，有可能也会发生错误，具体情况具体分析吧*
```xml
<!--注意这个依赖会有架包冲突，所以必须按照下面的方式排除几个架包哈，不然的话会报错如下-->
<!--An attempt was made to call the method org.eclipse.jetty.servlet.ServletMapping.setDefault(Z)V but i-->
<dependency>
	<groupId>org.apache.hive</groupId>
	<artifactId>hive-jdbc</artifactId>
	<version>1.2.1</version>
	<exclusions>
	<exclusion>
		<groupId>org.eclipse.jetty.aggregate</groupId>
		<artifactId>jetty-all</artifactId>
	</exclusion>
	</exclusions>
</dependency>

<!--用这一个包就可以了-->
<dependency>
	<groupId>org.apache.hive</groupId>
	<artifactId>hive-exec</artifactId>
	<version>1.2.1</version>
</dependency>

<!--注意：上面的hbase和hive的架包是有冲突的，最后各种查资料，查到这里，可以解决问题，先这么解决，后续需要学习一下如何通过查看源码解决jar包冲突的问题-->
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-common</artifactId>
	<version>2.6.0</version>
	<exclusions>
		<exclusion>
			<groupId>*</groupId>
			<artifactId>*</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```

* 01, 首先启动hiveserver2
  > hive/bin/hiveserver2 & 
  > 开启之后就多一个叫做RunJar的进程

* 02, 代码实现
  *代码比较多，这里不放*

## 11，yarn的内存限制

* 修改[yarn-site.xml]

```xml
<!--默认限制是开启的，最多分配给容器8G的物理内存，虚拟内存是物理内存的2.1倍。-->
<property>
	<name>yarn.nodemanager.resource.memory-mb</name>
	<value>8192</value>
</property>

<!--这里是关闭最大内存限制-->
<property>
	<name>yarn.nodemanager.pmem-check-enabled</name>
	<value>false</value>
</property>

<property>
	<name>yarn.nodemanager.vmem-check-enabled</name>
	<value>false</value>
</property>

<property>
	<name>yarn.nodemanager.vmem-pmem-ratio</name>
	<value>2.1</value>
</property>
```