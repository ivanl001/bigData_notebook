## 1，phoenix的安装
* 01，下载指定版本的phoenix
* 02，复制phoenix解压缩包中的phoenix-4.14.1-HBase-1.2-server.jar到hbase的lib目录下，注意：每个hbase包括slave节点上的都需要哈
* 03，需要重启hbase
* 04，通过phoenix的如下命令进行连接，后面跟的是zk的服务器哈
  > ./sqlline.py slave01

## 2, phoenix的使用
* 01, 显示table
  > !table

* 02, 创表,不支持Unsupported sql type: INT，所以用varchar即可
  > !sql create table test(id varchar(20) primary key, name varchar(20));

  > create table IF NOT EXISTS test.Person (IDCardNum INTEGER not null primary key, Name varchar(20),Age INTEGER);

* 03, 查看描述
  > !desc test;

* 04, 删除表
  > !sql drop table test;

* *在hbase中可以查看相关的信息*
  > list_namespace_tables 'default'
  > desc 'TEST'

* 05, 插入
  > upsert into test.Person (IDCardNum,Name,Age) values (103,'小王',22);

* 06, 查询
  > select * from test;

* 07, 删除
  > delete from test.person where age = 15

* 08, 聚合
  > select count(*) from  test.person;

## 3, squirrel的安装
* 01，我本人用的mac，下载jar安装包不能用，只能下载zip的安装包，可以运行包中的squirrel-sql.sh启动

* 02, Remove prior phoenix-[oldversion]-client.jar from the lib directory of SQuirrel, copy phoenix-[newversion]-client.jar(这个是phoenix解压缩中的包哈) to the lib directory (newversion should be compatible with the version of the phoenix server jar used with your HBase installation)
* 03, Start SQuirrel and add new driver to SQuirrel (Drivers -> New Driver)
* 04, In Add Driver dialog box, set Name to Phoenix, and set the Example URL to jdbc:phoenix:localhost.
* 05, Type “org.apache.phoenix.jdbc.PhoenixDriver” into the Class Name textbox and click OK to close this dialog.
* 06, Switch to Alias tab and create the new Alias (Aliases -> New Aliases)
* 07, In the dialog box, Name: any name, Driver: Phoenix, User Name: anything, Password: anything
* 08, Construct URL as follows: jdbc:phoenix: zookeeper quorum server.（就是需要把url中的local改成自己的zk服务器地址） For example, to connect to a local HBase use: jdbc:phoenix:localhost
Press Test (which should succeed if everything is setup correctly) and press OK to close.
* 09, Now double click on your newly created Phoenix alias and click Connect. Now you are ready to run SQL queries against Phoenix.

## 4, squirrel的使用
* 安装好之后就按照navicat的方式使用就行，里面直接用sql方式查询等等都是ok的