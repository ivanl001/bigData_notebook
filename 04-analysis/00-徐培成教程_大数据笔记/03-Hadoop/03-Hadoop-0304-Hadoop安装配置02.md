## 1，namenode和datanode的数据存储位置

```xml

//namenode和datanode的最终存储位置是由hdfs-site.xml,默认的配置文件在架包中的文件是：hdfs-default.xml中, 他们都是引用变量${hadoop.tmp.dir}，
//所以由此可见${hadoop.tmp.dir}的位置最终决定我们数据的存储位置，而这个属性是在架包中源文件core-default.xml，或者我们可以更改core-site.xml文件即可实现更改数据存储位置

//---------------------hdfs-default.xml-----------------
<property>
  <name>dfs.namenode.name.dir</name>
  <value>file://${hadoop.tmp.dir}/dfs/name</value>
  <description>Determines where on the local filesystem the DFS name node
      should store the name table(fsimage).  If this is a comma-delimited list
      of directories then the name table is replicated in all of the
      directories, for redundancy. </description>
</property>

<property>
  <name>dfs.datanode.data.dir</name>
  <!--当然如果有多个磁盘，这里可以以逗号隔开配置多个目录，前面的满了之后会自动从查找后面的-->
  <value>file://${hadoop.tmp.dir}/dfs/data</value>
  <description>Determines where on the local filesystem an DFS data node
  should store its blocks.  If this is a comma-delimited
  list of directories, then data will be stored in all named
  directories, typically on different devices. The directories should be tagged
  with corresponding storage types ([SSD]/[DISK]/[ARCHIVE]/[RAM_DISK]) for HDFS
  storage policies. The default storage type will be DISK if the directory does
  not have a storage type tagged explicitly. Directories that do not exist will
  be created if local filesystem permission allows.
  </description>
</property>


//---------------------core-default.xml，可更改core-site.xml进行覆盖-----------
<property>
  <name>hadoop.tmp.dir</name>
  <!--更改这个目录即可达到改变namenode和datanode的位置-->
  <value>/tmp/hadoop-${user.name}</value>
  <description>A base for other temporary directories.</description>
</property>

```

c

## 3，节点的服役和退役
*首先先说说明一下，slave文件并不是datanode连接名称节点的文件，这个slave文件只是开启的时候通过调用slave中的host开启这些数据节点，真正的判断是否会连接到名称节点，需要看dfs.hosts和dfs.hosts.exclude两个属性所对应的文件里面的内容，所以正规的服役或者退役的流程必须要先按照下面的配置文件创建好文件，然后配置到hdfs-site.xml中才行，下面的两个文件可以只在namenode节点上有即可*
```xml
<property>
  <name>dfs.hosts</name>
  <value>/usr/local/hadoop/etc/hadoop/hdfs.hosts.include</value>
  <description>这里配置文件地址，文件中的host是被允许连接到namenode节点，如果没有配置，那么所有到host地址都可以连接到名称节点上</description>
</property>

<property>
  <name>dfs.hosts.exclude</name>
  <value>/usr/local/hadoop/etc/hadoop/hdfs.hosts.exclude</value>
  <description>这里配置文件地址，文件中到host是不能连接到名称节点上</description>
</property>
```

### 3.1，退役过程
* 01，首先需要把需要退役的host添加到hdfs.hosts.exclude中去
* 02，刷新节点
  
  > hdfs dfsadmin -refreshNodes
* 03, 等待备份文件成功后
* 04，看webUI界面上提示Decommissioned后，从hdfs.hosts.include删除退役的那个host
* 05，重新刷新节点，退役完成
  
  > hdfs dfsadmin -refreshNodes

### 3.2, 服役节点，就是退役过程的反过程（整个过程是在已经启动了datanode和nodemanager的情况下进行的）
* 01，首先把服役节点添加到hdfs.hosts.exclude
* 02，刷新节点
* 03，然后添加节点到hdfs.hosts.include
* 04，刷新节点，看页面提示
* 05，从hdfs.hosts.exclude中移除要服役到节点
* 06，刷新节点

```shell
节点的服役和退役(hdfs)
----------------------
	[添加新节点]
	1.在dfs.include文件中包含新节点名称,该文件在nn的本地目录。
		[白名单]
		[s201:/soft/hadoop/etc/dfs.include.txt]
		s202
		s203
		s204
		s205
	2.在hdfs-site.xml文件中添加属性.
		<property>
			<name>dfs.hosts</name>
			<value>/soft/hadoop/etc/dfs.include.txt</value>
		</property>

	3.在nn上刷新节点
		$>hdfs dfsadmin -refreshNodes

	4.在slaves文件中添加新节点ip(主机名)
		s202
		s203
		s204
		s205		//新添加的

	5.单独启动新的节点中的datanode
		[s205]
		$>hadoop-daemon.sh start datanode

		
	[退役]
	1.添加退役节点的ip到黑名单,不要更新白名单.
		[/soft/hadoop/etc/dfs.hosts.exclude.txt]
		s205

	2.配置hdfs-site.xml
		<property>
			<name>dfs.hosts.exclude</name>
			<value>/soft/hadoop/etc/dfs.hosts.exclude.txt</value>
		</property>

	3.刷新nn节点
		$>hdfs dfsadmin -refreshNodes

	4.查看webui,节点状态在decommisstion in progress.

	5.当所有的要退役的节点都报告为Decommissioned,数据转移工作已经完成。

	6.从白名单删除节点,并刷新节点
		[s201:/soft/hadoop/etc/dfs.include.txt]
		...

		$>hdfs dfsadmin -refreshNodes

	7.从slaves文件中删除退役节点
	
	
节点的服役和退役(yarn)
----------------------
	[添加新节点]
	1.在dfs.include文件中包含新节点名称,该文件在nn的本地目录。
		[白名单]
		[s201:/soft/hadoop/etc/dfs.include.txt]
		s202
		s203
		s204
		s205
	2.在yarn-site.xml文件中添加属性.
		<property>
			<name>yarn.resourcemanager.nodes.include-path</name>
			<value>/soft/hadoop/etc/dfs.include.txt</value>
		</property>

	3.在nn上刷新节点
		$>yarn rmadmin -refreshNodes

	4.在slaves文件中添加新节点ip(主机名)
		s202
		s203
		s204
		s205		//新添加的

	5.单独启动新的节点中的nodemananger
		[s205]
		$>yarn-daemon.sh start nodemananger

		
	[退役]
	1.添加退役节点的ip到黑名单,不要更新白名单.
		[/soft/hadoop/etc/dfs.hosts.exclude.txt]
		s205

	2.配置yarn-site.xml
		<property>
			<name>yarn.resourcemanager.nodes.exclude-path</name>
			<value>/soft/hadoop/etc/dfs.hosts.exclude.txt</value>
		</property>

	3.刷新rm节点
		$>yarn rmadmin -refreshNodes

	4.查看webui,节点状态在decommisstion in progress.

	5.当所有的要退役的节点都报告为Decommissioned,数据转移工作已经完成。

	6.从白名单删除节点,并刷新节点

		$>yarn rmadmin -refreshNodes

	7.从slaves文件中删除退役节点
```







