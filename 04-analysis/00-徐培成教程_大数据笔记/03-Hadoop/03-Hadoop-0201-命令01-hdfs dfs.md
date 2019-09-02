

```java
如果通过直接 ssh slave01 jps这种方式调用jps报错jps命令找不到的话，只需要把jdk的bin中的jps文件创建了链接到/usr/bin目录下即可，这个是系统的原因，不需要纠结

上面的问题是ssh到对方，会使用新的bash环境，新的bash环境是用~/.bashrc 文件来加载环境变量的，除了进行软链接外，还可以把/etc/profile中的jdk的配置添加一份到~/.bashrc下也可以

ln -s /usr/local/jdk1.8.0_65/bin/jps /usr/bin/jps
```



# 1, hadoop的基本命令之hdfs dfs

## 1.0, hadoop fs 和hdfs dfs

* 两个命令其实一模一样，只需要把下面的hdfs dfs换成hadoop fs就可以完全执行下面的所有命令

```shell
hadoop fs ...
hdfs dfs ...
```



## 1.1，显示文件列表

```shell
# 不指定目录的情况下默认是用户目录下面的文件列表, 当前是在root用户下，就是/user/root, 当前用户是hive，就是/user/hive目录下
hdfs dfs -ls

# 指定具体目录
hdfs dfs -ls /user/root

# 递归形式显示所有目录下的文件或者文件夹
hdfs dfs -ls -R /user/root/

# 如果不用hdfs dfs而是用hadoop fs,如下
hadoop fs -ls /
```



## 1.2，上传文件

* put 和copyFromLocal是一样的

```shell
# 以下两种都是copy
hdfs dfs -put /root/ivanl001/ivanl001.txt /user/ivanl001/
hdfs dfs -copyFromLocal /root/ivanl001/ivanl002.txt /user/ivanl001/
```



## 1.3, 下载文件

```shell
hdfs dfs -get /user/ivanl001/ivanl002.txt /root/ivanl002/
```



## 1.4, 删除文件

```shell
# 注意：这个删除是移到垃圾箱，可以找出来的
hdfs dfs -rm -r -f /user/ivanl001/ivanl002.txt

# 如下是删除的时候会打印的信息
# 19/08/11 17:14:41 INFO fs.TrashPolicyDefault: Moved: 'hdfs://nameservice1/user/ivanl001/ivanl002.txt' to trash at: hdfs://nameservice1/user/root/.Trash/Current/user/ivanl001/ivanl002.txt
```



## 1.5, 创建目录

```shell
# 迭代创建目录
hdfs dfs -mkdir -p /user/ivanl001/ivanl002/ivanl003/
```



## 1.6, 追加本地文件到hdfs文件

* cdh下命令行不能直接追加，会报错，不知道是不是因为搭建的问题，先放着

```shell
# 可能出现权限不允许，可以参照下面的修改权限
hdfs dfs -appendToFile /root/ivanl001/ivanl001.txt /user/ivanl001/ivanl001
```



## 1.7, 查看命令

```shell
hdfs dfs -cat /user/ivanl001/ivanl001
```



## 1.8, 修改权限

```shell
# 修改ivanl001文件的权限，给所有人增加写入权限
# 注意：修改权限的时候需要在对应的用户目录下修改，也就是在文件所有者名下修改
hdfs dfs -chmod a+w /user/ivanl001/ivanl001
```




> SPOF: single point of failure



