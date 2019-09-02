# 2，hdfs的其他命令



## 2.0, hdfs dfs

```shell
hdfs dfs -ls /
# 这个和hadoop fs -ls /完全等价
```



## 2.1,  hdfs getconf -confKey

* 通过这个命令获取某些属性的值，比如说：
  

```shell
hdfs getconf -confKey dfs.namenode.fs-limits.min-block-size
```



## 2.2, hdfs oiv : offline fsimage viewer

>  镜像文件是二进制文件，所以不能直接看
>
> 镜像文件主要描述文件结构，权限，修改时间，副本数， 块信息
>
> ⚠️：文件夹是没有副本数的，可以参考hdfs 50070页面上，文件夹的副本数是0



### 转换成xml文件后查看

```shell
mkdir -p /root/test/
touch /root/test/fsimage01.xml

hdfs oiv -i fsimage_0000000000000085226 -o /root/test/fsimage01.xml -p XML
```



### 直接查看

```shell
hdfs oiv -i fsimage_0000000000000085226

# 这里是开启一个web服务器，但是我不知道怎么看，先放着
```





## 2.3, hdfs oev :  offline edit viewer

> 查看最近新进行的编辑的信息, 会定期的融合到镜像文件中

* 好像是不能直接-i访问，只能-o连带输出，如下
  

```shell
touch /root/test/edit01.xml

hdfs oev -i edits_0000000000000086575-0000000000000086609 -o /root/test/edit01.xml -p XML
```





## 2.4, hdfs dfsadmin

>  这个是管理员操作的一个命令集
>
> 比如进入安全模式，滚动编辑日志，创建删除快照等等



### 2.4.0, 滚动编辑日志

```shell
hdfs dfsadmin -rollEdits
```



### 2.4.1, 安全模式

```shell
hdfs dfsadmin -safemode enter
# 查看状态
hdfs dfsadmin -safemode get
hdfs dfsadmin -safemode leave
# 写上这一句，后面的命令如果遇到安全模式， 会等待，而不是直接停止
hdfs dfsadmin -safemode waite
```



### 2.4.2, 保存命名空间

> 相当于把镜像文件和编辑日志手动融合

```shell
# 需要先进入安全模式
hdfs dfsadmin -saveNamespace
```



### 2.4.3, 设置名称配额

* 这个是文件夹内名字的限制
* 比如说设置成1，那么就完全不能再创建新的文件夹，因为唯一的配额已经被本身占用了
* 这个是递归的

```shell
hdfs dfsadmin -help setQuota


-setQuota <quota> <dirname>...<dirname>: Set the quota <quota> for each directory <dirName>.
                The directory quota is a long integer that puts a hard limit
                on the number of names in the directory tree
                For each directory, attempt to set the quota. An error will be reported if
                1. N is not a positive integer, or
                2. User is not an administrator, or
                3. The directory does not exist or is a file.
                Note: A quota of 1 would force the directory to remain empty.

```



### 2.4.3, 设置空间配置，同上

* 这个是文件夹内空间的限制
* 包括副本

```shell
hdfs dfsadmin -help setSpaceQuota
```



```shell
配额管理(quota)
-------------------
	[目录配额]
	计算目录下的所有文件的总个数。如果1，表示空目录。
	$>hdfs dfsadmin -setQuota 1 dir1 dir2		//设置目录配额
	$>hdfs dfsadmin -clrQuota 1 dir1 dir2		//清除配额管理

	[空间配额]
	计算目录下的所有文件的总大小.包括副本数.
	空间配置至少消耗384M的空间大小(目录本身会占用384M的空间)。
	$>hdfs dfsadmin -setSpaceQuota 3 data
	$>echo -n a > k.txt
	$>hdfs dfs -put k.txt data2
	$>hdfs dfsadmin -clrSpaceQuota dir1			//清除配额管理

```





## 2.5, hdfs dfs

### 2.5.1, 快照

这个dfs命令集是最常用的，其他的就慢慢研究，这里就介绍下创建快照

快照不会产生新的文件，修改原文件也不会影响快照的内容，是使用差值存储

* 为文件夹开启快照功能
  

```shell
hdfs dfsadmin -allowSnapshot /user/root/test
```

* 创建快照
  

```shell
hdfs dfs -createSnapshot /user/root/test
```

* 删除快照,快照名在创建快照的时候会给出
  

```shell
hdfs dfs -deleteSnapshot /user/root/test s20181020-145049.152
```

