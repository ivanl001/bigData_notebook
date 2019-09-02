#### 1, 虚拟机安装

#### 2， 环境准备

> 注意：server节点至少是8G内存，8G也很少，后续会有很多问题
>
> 如果是通过虚拟机搭建，16G的勉强能搭建起来，但是后续问题会非常多，server至少8G，agent也需要4G或者8G，再少 就不要玩了

##### 2.1, ssh免密登录

* 把公钥文件添加到authorized_keys文件中即可

##### 2.2， 远程同步脚本

###### imrsync.sh:

```shell
#! /bin/bash

# 脚本作用：同步文件或者文件夹

if(($#!=1)); then echo '请输入正确的需要同步的文件夹或文件的路径';exit;fi;

echo;

path=$@;

relativeDirPath=`dirname $path`;#这里取出来有时候是.,也就是当前目录
cd $relativeDirPath;
dirPath=`pwd`;

basePath=`basename $path`;

fullPath=$dirPath/$basePath;

echo '--------------------------------同步源目录开始:'$fullPath'------------------------------------';

for((i=1;i<4;i++));
do
rsync -avl $@ root@node20$i:$fullPath;
done;

echo '--------------------------------同步源目录结束:'$dirPath/$basePath'------------------------------------';

echo;
```

##### 2.3, 远程命令脚本编写

###### imcall.sh

```shell
#! /bin/bash

#if((n!=1)); then echo '请输入正确命令';exit;fi;#这里不能判断一个参数，不然ls -a这种就会有问题

echo;#打印一行空格

echo '---------------------------------执行命令开始:'$@'-----------------------------------';

echo;#打印一行空格

#echo '---------------master---------------'
#ssh localhost "$@";

for((i=1;i<4;i++))
do
echo '---------------'node20$i'--------------';
ssh node20$i "$@";
done;

echo;

echo '---------------------------------执行命令结束:'$@'-----------------------------------';
```

#####  2.4, jdk安装

* 略过

##### 2.5， mysql安装

*现在已经用MariaDB完全取代msyql，而是使用方法完全一致，并且安装更方便一些，安装方法如下：*

- 01， 使用yum直接安装MariaDB

  > yum -y install mariadb*

- 02, 开启服务，并设置开机启动

  > systemctl start mariadb.service
  > systemctl enable mariadb.service

- 03, 尝试登陆,刚开始是没有密码的，应该可以直接登陆成功

  > mysql

- 04，如果成功之后，退出，设置密码

  > exit;
  > mysql_secure_installation;

- 05，提示输入密码，如果是刚装的，密码是空，直接enter即可，然后输入用户和密码，按照提示操作即可

- 06, 都成功之后，可以按照正常流程进行登陆即可

  > mysql -u root -p *****   //root是你设置的用户名

- 07， 默认情况下，外部是不能直接连接我们的数据库的，需要更改设置,并刷新权限

  - 这里是可以设置不同的用户名和不同的密码的哦，好强大的样子

  > GRANT ALL PRIVILEGES ON \*.\* TO 'root'@'%'IDENTIFIED BY ',.' WITH GRANT OPTION;
  >
  > 
  >
  > flush privileges;

- 08, 可以从用户表中查看root访问权限

  > select host, user from mysql.user;

##### 2.6， 同步时间

* 略

##### 2.6.1、关闭selinu

```
命令：vi /etc/sysconfig/selinux
注释掉SELINUX=enforcing
添加SELINUX=disabled
查看状态：/usr/sbin/sestatus -v
```

##### 2.7， 安装依赖包

```shell
yum install chkconfig python bind-utils psmisc libxslt zlib sqlite cyrus-sasl-plain cyrus-sasl-gssapi fuse fuse-libs redhat-lsb -y
```



#### 3, CDH manager安装

###### 3.1， CDH安装相关的包

- cloudera manager包 ：5.7.2 cloudera-manager-centos7-cm5.7.2_x86_64.tar.gz 
  下载地址：<http://archive.cloudera.com/cm5/cm/5/cloudera-manager-centos7-cm5.7.2_x86_64.tar.gz>

- CDH-5.7.2-1.cdh5.7.2.p0.18-el7.parcel

  下载地址：http://archive.cloudera.com/cdh5/parcels/5.7.2/

  注意：需要下载parcel文件，sha1文件和manifest.json文件

###### 3.2，先添加用户，linux下直接执行即可

```shell
useradd --system --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm
```

###### 3.4，解压文件，并分发到每个机器上

```shell
# 解压文件
mkdir /opt/cloudera-manager
tar -zxvf cloudera-manager-centos7-cm5.7.2_x86_64.tar.gz -C /opt/cloudera-manager
```

###### 3.5, 修改配置，注意：注意注意注意：这个server_host是主机的host，所有的agent都需要配置成一样的哈！！！！

- 修改/opt/cloudera-manager/cm-/etcc/

- vim /opt/cloudera-manager/cm-5.7.2/etc/cloudera-scm-agent/config.ini

  ```shell
  # Hostname of the CM server.
  server_host=node201
  ```

###### 3.6， jdbc的连接

* mysql-connector -java-5.1.26-bin.jar
* 把mysql-connector-java-5.1.26-bin.jar拷贝到/usr/share/java/目录下，并更改名字为：mysql-connector-java.jar

###### 3.7, 配置CM Server数据库(应该只在主节点即可)

```mysql
grant all on *.* to 'cdh'@'%' identified by 'cdh' with grant option;

# 然后到/opt/cloudera-manager/cm-5.7.2/share/cmf/schema目录下，执行如下命令：
/opt/cloudera-manager/cm-5.7.2/share/cmf/schema/scm_prepare_database.sh mysql cdh -hnode201 -ucdh -pcdh --scm-host node201 scm scm scm
```

###### 3.8，创建Parcel目录 

*也就是各种组件的仓库*

* server节点

  ```shell
  mkdir -p /opt/cloudera/parcel-repo
  chown cloudera-scm:cloudera-scm /opt/cloudera/parcel-repo
  ```

* Agent节点

  ```shell
  mkdir -p /opt/cloudera/parcels
  chown cloudera-scm:cloudera-scm /opt/cloudera/parcels
  ```

###### 3.9, 制作CDH本地源

*把上面下载parcel文件，sha1文件和manifest.json文件放到server节点的/opt/cloudera/parcel-repo下*



###### 3.10,  完了之后， 在Server节点上开启server

> /opt/cloudera-manager/cm-5.7.2/etc/init.d/cloudera-scm-server start

###### 3.11, 在agent节点上开启agent

> /opt/cloudera-manager/cm-5.7.2/etc/init.d/cloudera-scm-agent start

###### 3.12, 配置客户端电脑的host文件

*添加如下内容*

```
# 这些是虚拟机的配置，学习用
192.168.83.101  node101
192.168.83.201  node201
192.168.83.202  node202
192.168.83.203  node203
```

###### 3.13, 浏览器访问

> http://node201:7180/

* 账号：admin
* 密码：admin

###### 3.14， 问题01：

>  就是server监控不到agent，在agent查看log，发现没起起来， 然后查看状态，如下：
>
> cloudera-scm-agent dead but pid file exist

* 后来发现是agent节点的hosts文件没有更新，更新之后正常

###### 3.15， 问题02：

>  之前的操作在分配parcel的时候回出现问题

* 解决办法：
* 把/opt/cloudera/parcel-repo中的CDH-5.7.2-1.cdh5.7.2.p0.18-el7.parcel.sha1文件改名为CDH-5.7.2-1.cdh5.7.2.p0.18-el7.parcel.sha，去掉后面的1

###### 3.16, 问题03

> 客户端配置 (id=2) 已使用 1 退出,而预期值为 0

参考：https://blog.csdn.net/clerk0324/article/details/73611937

```
1.首先我们需要找到此处日志目录，并不是/opt/cm-5.5.0/log。针对使用tar.gz包进行离线安装的目录，日志应该在：/opt/cm-5.5.0/run/cloudera-scm-agent/process/ccdeploy_spark-conf_etcsparkconf.cloudera.spark_-6842105649195360849/logs，因为我是在spark这一步进行部署客户端配置失败的时候出错的，所以就找的spark这一文件夹下的日志。如果是使用.bin包安装的，则有可能是在/var/run/cloudera-scm-agent/process/目录下。找到日志文件之后，

你应该能在日志文件中找到：export JAVA_HOME=/usr/java/default、JAVA_HOME=/usr/java/default、Error: JAVA_HOME is not set and could not be found等关键词，

所以明确了是jdk没有装好，为什么没装好，因为我的是使用tar.gz的jdk包安装的，没有往/usr/java中添加软链接，而这里默认是去/usr/java/default中找环境变量，才会报找不到java_home。

安装jdk的方法:把jdk软连接到/usr/java/default首先查看是否有/usr/java目录，没有的话新建此目录：mkdir /usr/java。然后添加软连接到/usr/java/default，命令如下:ln -s /home/monitor/apps/jdk1.7.0_45 /usr/java/default
为什么要添加软连接到/usr/java/default?这是因为有些软件，不会去找环境变量的java，而是找/usr/java下的，比如说cloudera manager在部署最后的spark时，一直报“上的客户端配置 (id=3) 已使用 1 退出,而预期值为 0”这个错误，其中一个原因就是访问java访问不到，参考：cloudera manager报错“客户端配置 (id=3) 已使用 1 退出,而预期值为 0”

2.如果还是不行，但是又找不到相关错误，就该考虑是不是内存不足导致没法将配置部署到客户端了。

3.编辑/etc/sudoers，在root后面添加如下内容：cloudera-scm    ALL=(ALL)    NOPASSWD:ALL。这一步没有验证，不知道有什么影响
--------------------- 
作者：两榜进士 
来源：CSDN 
原文：https://blog.csdn.net/clerk0324/article/details/73611937 
版权声明：本文为博主原创文章，转载请附上博文链接！
```

###### 3.17, 问题04

> 查看日志报错是：ImportError: libxslt.so.1: cannot open shared object file: No such file or directory

* 解决办法：
* 原因是centos缺少库文件，执行如下命令即可
* yum install krb5-devel cyrus-sasl-gssapi cyrus-sasl-deve libxml2-devel libxslt-devel mysql mysql-devel openldap-devel python-devel python-simplejson sqlite-devel -y



