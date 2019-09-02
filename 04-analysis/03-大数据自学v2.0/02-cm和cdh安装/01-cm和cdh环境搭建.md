>  本文是在00-服务器搭建基础之上搭建的

#### 01-cm和cdh环境搭建

* cm是Cloudera Manager，cdh是Cloudera Distributed Hadoop。前者是管理后者的一个平台，后者是Hadoop的一个发行版本。

！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！

注意：安装时候的安装包最好现下，有些安装包经过复制拷贝，有可能是会改变默认的文件夹的权限，这样到时候就会有问题。我安装的时候





##### 1, 关闭selinu

> vim /etc/sysconfig/selinux

```
SELINUX=disabled
```

* 重启启动服务器reboot生效

*  查看状态：

  > /usr/sbin/sestatus -v

##### 2, 安装依赖包

```shell
yum install chkconfig python bind-utils psmisc libxslt zlib sqlite cyrus-sasl-plain cyrus-sasl-gssapi fuse fuse-libs redhat-lsb -y

# 下面这些先不装
yum install krb5-devel cyrus-sasl-gssapi cyrus-sasl-deve libxml2-devel libxslt-devel mysql mysql-devel openldap-devel python-devel python-simplejson sqlite-devel -y
```



##### 3, cm安装

> !!!下次可以安装5.12.1, 尚学堂大数据数仓中都有提供

###### 3.1， 下载cm相关包和吃的需要的parcels文件

- cloudera manager包 ：5.7.2 cloudera-manager-centos7-cm5.7.2_x86_64.tar.gz 
  下载地址：<http://archive.cloudera.com/cm5/cm/5/cloudera-manager-centos7-cm5.7.2_x86_64.tar.gz>

- CDH-5.7.2-1.cdh5.7.2.p0.18-el7.parcel

  下载地址：http://archive.cloudera.com/cdh5/parcels/5.7.2/

  注意：需要下载parcel文件，sha1文件和manifest.json文件，

  http://archive.cloudera.com/cdh5/parcels/
  
  
  
  http://archive.cloudera.com/cm5/cm/5/
  
  

###### 3.2, 解压文件，并分发到每个机器上

```shell
# 解压文件
mkdir /opt/cloudera-manager
tar -zxvf cloudera-manager-centos7-cm5.7.2_x86_64.tar.gz -C /opt/cloudera-manager
# 分发
imrsync.sh /opt/cloudera-manager/
```



###### 3.3, 添加用户cm用户，linux下直接执行即可

```shell
useradd --system --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm

# 查看用户是否创建ok
cat /etc/passwd
cat /etc/passwd 可以查看所有用户的列表
w 可以查看当前活跃的用户列表
cat /etc/group 查看用户组

groups 查看当前登录用户的组内成员
groups gliethttp 查看gliethttp用户所在的组,以及组内成员
whoami 查看当前登录用户名

一个简明的layout命令

cat /etc/passwd|grep -v nologin|grep -v halt|grep -v shutdown|awk -F":" '{ print $1"|"$3"|"$4 }'|more
```



###### 3.4, 修改cm的server服务器配置

>  注意注意注意：这个server_host是主机的host，所有的agent都需要配置成一样的哈！！！！

- vim /opt/cloudera-manager/cm-5.7.2/etc/cloudera-scm-agent/config.ini

- vim /opt/cloudera-manager/cm-5.12.1/etc/cloudera-scm-agent/config.ini

- vim /opt/cloudera-manager/cm-5.16.2/etc/cloudera-scm-agent/config.ini

  ```shell
  # Hostname of the CM server.
  server_host=centos01
  ```



###### 3.5,  jdbc的连接依赖

* 这一步之需要在server节点做就可以了，agent节点不需要

- 下载mysql-connector -java-5.1.26-bin.jar
- 把mysql-connector-java-5.1.26-bin.jar拷贝到/usr/share/java/目录下，并更改名字为：mysql-connector-java.jar
- ⚠️注意：这里每个节点都要有，不然后面hue验证的时候可能会出现验证不成功的情况！！！！

> mkdir -p /usr/share/java
>
> mv mysql-connector-java.jar /usr/share/java

###### 3.6, 配置CM Server数据库(应该只在主节点即可)

```mysql
grant all on *.* to 'cdh'@'%' identified by 'cdh' with grant option;

# 要刷新权限
flush privileges;

# 然后到/opt/cloudera-manager/cm-5.7.2/share/cmf/schema目录下，执行如下命令：
/opt/cloudera-manager/cm-5.7.2/share/cmf/schema/scm_prepare_database.sh mysql cdh -h centos01 -u cdh -p cdh --scm-host centos01 scm scm scm

-h database host
-u database username
-p database password
--scm-host scm server hostname

/opt/cloudera-manager/cm-5.12.1/share/cmf/schema/scm_prepare_database.sh mysql cdh -h centos01 -u cdh -p cdh --scm-host centos01 scm scm scm
# 注意：如果让输入密码，就是输入cdh，因为上面赋予权限的时候就是cdh
/opt/cloudera-manager/cm-5.16.2/share/cmf/schema/scm_prepare_database.sh mysql cdh -h centos01 -u cdh -p cdh --scm-host centos01 scm scm scm
```



###### 3.7, 创建Parcel目录,更改权限等

*也就是各种组件的仓库*

- server节点

  ```shell
  mkdir -p /opt/cloudera/parcel-repo
  chown cloudera-scm:cloudera-scm /opt/cloudera/parcel-repo
  ```

- Agent节点

  ```shell
  mkdir -p /opt/cloudera/parcels
  chown cloudera-scm:cloudera-scm /opt/cloudera/parcels
  ```



###### 3.8, 制作CDH本地源

> 注意注意注意：上面下载的sha1文件需要改名后缀名sha后放到server节点的/opt/cloudera/parcel-repo下目录下才行哈！！！！只放server 节点即可

*把上面下载parcel文件，sha文件和manifest.json文件放到server节点的/opt/cloudera/parcel-repo下*



##### 4, cm的启动

###### 4.1, 在Server节点上开启server

> /opt/cloudera-manager/cm-5.7.2/etc/init.d/cloudera-scm-server start
>
> /opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-server start
>
> /opt/cloudera-manager/cm-5.16.2/etc/init.d/cloudera-scm-server start

```java
cm的sever的日志目录位置：
/opt/cloudera-manager/cm-5.7.2/log/cloudera-scm-server/cloudera-scm-server.log 
/opt/cloudera-manager/cm-5.12.1/log/cloudera-scm-server/cloudera-scm-server.log 
/opt/cloudera-manager/cm-5.16.2/log/cloudera-scm-server/cloudera-scm-server.log
```

* 成功后，jps会多处一个main进程，日志中无报错

###### 4.2, 在agent节点上开启agent

> /opt/cloudera-manager/cm-5.7.2/etc/init.d/cloudera-scm-agent start
>
> /opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent start
>
> /opt/cloudera-manager/cm-5.16.2/etc/init.d/cloudera-scm-agent start

```java
cm的agent的日志目录位置：
/opt/cloudera-manager/cm-5.7.2/log/cloudera-scm-agent/cloudera-scm-agent.log
/opt/cloudera-manager/cm-5.12.1/log/cloudera-scm-agent/cloudera-scm-agent.log

/opt/cloudera-manager/cm-5.16.2/log/cloudera-scm-agent/cloudera-scm-agent.log
```



###### 4.3, 浏览器访问验证

> http://centos01:7180/

- 账号：admin
- 密码：admin



###### 4.4, 可能出现的问题

```java
-- 可能出现的问题：

1， 查看日志报错是：ImportError: libxslt.so.1: cannot open shared object file: No such file or directory

- 解决办法：
- 原因是centos缺少库文件，执行如下命令即可
- yum install krb5-devel cyrus-sasl-gssapi cyrus-sasl-deve libxml2-devel libxslt-devel mysql mysql-devel openldap-devel python-devel python-simplejson sqlite-devel -y

2，如果需要重新安装，注意删除mysql库中的内容哈

```

