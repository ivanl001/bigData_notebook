* 一些配置参考，一些问题等等

https://blog.csdn.net/u011026329/article/details/79167722



a关键步骤直接截图内容表示，有些没有表示的部分随意选择即可

* 如果有图中哪些和你的不一样，说明之前的步骤应该有问题，排查一下

##### 1, 选择Cloudera Express

![image-20190510020524205](assets/image-20190510020524205.png)



##### 2, 选择当前管理的主机

这里如果不一样，说明之前的步骤应该是有问题的，先排查一下吧！

![image-20190510020645191](assets/image-20190510020645191.png)



##### 3, 选择之前放入到parcel-repo文件夹到parcel版本离线安装

![image-20190510020754801](assets/image-20190510020754801.png)



##### 4, 安装进度

![image-20190510021322450](assets/image-20190510021322450.png)



##### 5, 验证检查，如果问题，按提示解决

![image-20190510021436054](assets/image-20190510021436054.png)



##### 6, 默认有两个警告解决办法

###### 6.1, 警告1：swappiness

**Cloudera 建议将 /proc/sys/vm/swappiness 设置为最大值 10。当前设置为 30**--解决方法：

* 临时修改:

```shell
sysctl vm.swappiness=10
cat /proc/sys/vm/swappiness
```

这里我们的修改已经生效，但是如果我们重启了系统，又会变成60.

**永久修改：**

在`/etc/sysctl.conf` 文件里添加如下参数：

```shell
vm.swappiness=10
```

或者：

```shell
echo 'vm.swappiness=10'>> /etc/sysctl.conf
```





###### 6.2, 警告2：透明大页面压缩

**已启用透明大页面压缩,可能会导致重大性能问题。请运行“echo never > /sys/kerne**—解决办法：

```shell
echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo never > /sys/kernel/mm/transparent_hugepage/enabled 
```



##### 7, 选择CDH的组合

重新检查后没有警告了，然后继续往下走：

![image-20190510022532394](assets/image-20190510022532394.png)



##### 8, 角色分配

> 注意：Hbase Thrift Server最好选择和Hue相同服务器，如果这里不选择，后续如果需要用hue连接hbase，依然需要安装，具体看最后的警告解决部分

![image-20190510022747658](assets/image-20190510022747658.png)



##### 9, 选择zk服务器

![image-20190510022849402](assets/image-20190510022849402.png)



##### 10, 集群hive和oozie的数据库设置

> 注意：页面中有数据库创建的链接，最好按照指定方式创建，要不后续oozie会出现问题，具体看后续错误解决部分

```sql
-- hive的创建数据库操作
create database hive default character set utf8;
grant all privileges on hive.* to 'hive'@'localhost' identified by 'hive';
grant all privileges on hive.* to 'hive'@'%' identified by 'hive';

-- hue的创建数据库操作
create database hue default character set utf8;
grant all privileges on hue.* to 'hue'@'localhost' identified by 'hue';
grant all privileges on hue.* to 'hue'@'%' identified by 'hue';



-- oozie的创建数据库操作，hive也按照这种方式来，如果不这样，后续会出现我后面oozie安装失败的错误哈
-- hive用root用户创建也ok，hive不报错的，但是oozie必须要用oozie用户创建
$ mysql -u root -p
Enter password:

mysql> create database oozie default character set utf8;
Query OK, 1 row affected (0.00 sec)

mysql>  grant all privileges on oozie.* to 'oozie'@'localhost' identified by 'oozie';
Query OK, 0 rows affected (0.00 sec)

mysql>  grant all privileges on oozie.* to 'oozie'@'%' identified by 'oozie';
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye

-- 然后需要拷贝/usr/share/java/mysql-connector-java.jar到/var/lib/oozie/目录下
```



![image-20190510023119798](assets/image-20190510023119798.png)



##### 11, 默认配置即可

![image-20190510023309026](assets/image-20190510023309026.png)



##### 12, 安装进度

![image-20190510023403864](assets/image-20190510023403864.png)



##### 13, 错误解决1: oozie安装失败

![image-20190510024527857](assets/image-20190510024527857.png)



```sql

```



> 错误原因：10步中创建oozie库方式不对，具体解决步骤参看下图

出现这个问题是应为没有赋予oozie用户权限，重新回到前面的页面：

![image-20190510034136212](assets/image-20190510034136212.png)

根据页面链接， 我找到了那个章节：

https://www.cloudera.com/documentation/enterprise/5-7-x/topics/install_oozie_ext_db.html#concept_dkp_xh4_hp

具体内容如下：

###### Configuring MySQL for Oozie



###### Install and Start MySQL 5.x

See [MySQL Database](https://www.cloudera.com/documentation/enterprise/5-7-x/topics/cm_ig_mysql.html#cmig_topic_5_5).



###### Create the Oozie Database and Oozie MySQL User

For example, using the MySQL mysql command-line tool:

```
$ mysql -u root -p
Enter password:

mysql> create database oozie default character set utf8;
Query OK, 1 row affected (0.00 sec)

mysql>  grant all privileges on oozie.* to 'oozie'@'localhost' identified by 'oozie';
Query OK, 0 rows affected (0.00 sec)

mysql>  grant all privileges on oozie.* to 'oozie'@'%' identified by 'oozie';
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
```



###### Add the MySQL JDBC Driver JAR to Oozie

Copy or symbolically link the MySQL JDBC driver JAR into one of the following directories:

- For installations that use *packages*: /var/lib/oozie/
- For installations that use *parcels*: /opt/cloudera/parcels/CDH/lib/oozie/lib/

directory.

**Note:** You must manually download the MySQL JDBC driver JAR file.



###### **翻译总结：**

* 上面的内容很清楚了，主要有两点：

  * 1， 要创建数据库，并赋予oozie用户登陆权限

    ```sql
    create database oozie default character set utf8;
    grant all privileges on oozie.* to 'oozie'@'localhost' identified by 'oozie';
    grant all privileges on oozie.* to 'oozie'@'%' identified by 'oozie';
    ```

    

  * 2，拷贝mysql的connector的jar包到oozie的目录下(其实这步不做应该也可以，好像会是自已拷贝的)

![image-20190510035124887](assets/image-20190510035124887.png)



##### 14, 重跑即可

![image-20190510044930910](assets/image-20190510044930910.png)



##### 15，成功

![image-20190510044944217](assets/image-20190510044944217.png)





##### 16，警告解决

对了，默认安装可能在hue中用不了hbase, 因为Thrift  Server没有选择服务器，会有如下警告：

###### 16.1,  Hue警告: 必须在 HBase 服务中配置 Thrift Server 角色以使用 Hue HBase Browser 应用程序。

解决办法：

给HBase添加Thrift Server角色, 为了方便, 将Thrift Server添加到Hue同一主机

![image-20190510122935865](assets/image-20190510122935865.png)

![image-20190510123036117](assets/image-20190510123036117.png)

重启集群后警告会变成：在 HBase Thrift Server 属性中选择服务器以使用 Hue HBase Browser 应用程序

那么在hue的配置页面开启即可，如下图，重启！

![image-20190510130824381](assets/image-20190510130824381.png)



##### 17, 提示

###### 如果重新安装：一定要删除ysql中的cdh库哈！



##### 18, “大数据项目之电商数仓”其他配置

###### 18.1, 关闭hdfs权限检查，防止上传文件出错，不要随意关闭哈

![image-20190726190647231](assets/image-20190726190647231.png)

###### 18.2, 配置LZO压缩支持

* 具体参考教程或者教程中的文档

* https://www.cnblogs.com/zhzhang/p/5695027.html
* 这个是centos7上的配置地址哦
* http://archive.cloudera.com/gplextras5/parcels/5.12.1/

* 1, 首先在parcel设置中添加lzo的parcel地址
  * http://archive.cloudera.com/gplextras5/parcels/5.12.1/
  * ![image-20190803215642849](assets/image-20190803215642849.png)
* 2, 然后在parcel里下载，分配，激活
  * ![image-20190803215734042](assets/image-20190803215734042.png)
* 3, 配置hdfs支持lzo
  * io.compression.codecs
  * com.hadoop.compression.lzo.LzoCodec
  * com.hadoop.compression.lzo.LzopCodec
  * ![image-20190803215816526](assets/image-20190803215816526.png)
* 4, 配置yarn支持lzo
  * mapreduce.application.classpath
  * /opt/cloudera/parcels/GPLEXTRAS/lib/hadoop/lib/*
  * ![image-20190803220131727](assets/image-20190803220131727.png)
* 5, 修改MR应用程序环境变量
  * mapreduce.admin.user.env
  * /opt/cloudera/parcels/GPLEXTRAS/lib/hadoop/lib/native
  * ![image-20190803220405447](assets/image-20190803220405447.png)

* 6, 重启hdfs，yarn即可



##### 19, hue报错：“you are accessing a non-optimized Hue”

* 这个是因为hue的load balancer没有启动，参考20即可     



##### 20, CDH Hue Load Balancer 无法启动

https://ask.csdn.net/questions/671624

```shell
# 执行后重启hue即可
yum -y install httpd mod_ssl
```

