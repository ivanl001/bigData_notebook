*hbase是java语言写的，所以python并不能直接访问hbase，但是可以通过启Thrift服务器连接到hbase上，然后python通过jdbc协议访问thrift服务器进行访问hbase*

## 1，python通过Thrift2服务器访问hbase数据库
* 01，启动hbase的thrift2服务器,启动之前至少要保证hadoop和hbase已经启动
  *9090是rpc监听端口，9095是ui监听端口*
  > hbase-daemon.sh start thrift2


* 02, 通过如下ui地址查看是否启动成功
  http://master:9095

* 03，首先需要编译python的thrift模块，编译python模块需要先安装Thrift，windows上可以直接下载exe文件，mac上可以直接使用brew
  > brew install thrift

* 04, 编译python的thrift模块，需要先下载hbase的源文件包，hbase-2.1.0-src.tar.gz，解压后在如下文件夹下执行如下命令
  > 文件夹:/hbase-2.1.0/hbase-thrift/src/main/resources/org/apache/hadoop/hbase/thrift2下有一个hbase.thrift文件，按照下面命令编译即可。解释一下：其实还有一个thrift文件夹，这相当于是两套api，后者会更接近于hbase的api，所以使用的时候才会让我们手动编译出我们需要的api然后使用，我们这里用thrift2的api，所以编译对应文件夹下的文件即可

  > 编译命令:thrift -gen py hbase.thrift
  * 上面的命令的意思就是用刚才brew安装的thrift编译器编译hbase.thrift成python代码，完成之后会在这个目录下面生成一个gen-py文件夹，把里面的hbase文件夹拷贝到项目的python包中去，后面后买呢引用
  
* 05，开始写代码, 在写代码的时候还有一个可以通过idea自动安装的就是:python的thrift插件，这个写代码的时候会提示你安装。具体代码如下:
```python
#
# author      : ivanl001
# creator     : 2018-12-08 19:57
# description : 
# 

# 这里是通过idea自动下载的thrift的python插件
from thrift.protocol import TBinaryProtocol
from thrift.transport import TSocket, TTransport

# 这两行是我编译hbase.thrift后拷贝过来的hbase文件夹里面的代码哈
from mythrift.hbase import THBaseService
from mythrift.hbase.ttypes import *

# ！！！！！有一点需要注意：在测试的时候一定要保证hbase已经开启，thrift已经开启，就算hbase没开启，thrift也能正常开启，但是测试的时候会一直卡住，我测试的时候已经被坑死了！！！！
# 因为我是在master上执行的hbase-daemon.sh start thrift2，所以开启的是master的9090端口
transport = TSocket.TSocket("master", 9090)
# 可以设置超时
# transport.setTimeout(5000)
# 设置传输方式（TFramedTransport或TBufferedTransport）
transport = TTransport.TBufferedTransport(transport)
# 设置传输协议
protocol = TBinaryProtocol.TBinaryProtocol(transport)

# 确定客户端
client = THBaseService.Client(protocol)

# 打开连接
transport.open()


# tableName = "ns1:t1"
# rowkey = "r0001410"
# row = client.get(tableName,rowkey)
# print(row)

table = b'ns1:t1'
row = b'row1'
v1 = TColumnValue(b'f1', b'id', b'101')
v2 = TColumnValue(b'f1', b'name', b'102')
v3 = TColumnValue(b'f1', b'value', b'103')
values = [v1, v2, v3]
put = TPut(row, values)
client.put(table, put)

transport.close()
```

## 2，spark命令行中写python代码测试

* 00, 注意：如果提示imcluster等信息，是因为启动的单机程序却发现了分布式的配置文件，只需要移除hdfs-site.xml, core-site.xml, hive-site.xml，然后重新启动spark即可，启动方式如下

* 01, 首先启动spark的python的命令行界面
  > ./pyspark --master local[4]

* 02, 输入正常的python的程序
```python

# 案例01
# 组装数据
arr = ["ivanl001", "ivanl002", "zhang", "jack", "mike"]
# 转成rdd
rdd01 = sc.parallelize(arr)
# map成单词和长度的组合
rdd02 = rdd01.map(lambda e: (e, len(e)))
# collect查看
rdd02.collect()
# 结果如下:
# [('ivanl001', 8), ('ivanl002', 8), ('zhang', 5), ('jack', 4), ('mike', 4)]


# 案例02
rdd01 = sc.textFile("/root/ivanl001/test/zhang.txt")
rdd02 = rdd01.flatMap(lambda e : e.split(" "))
rdd03 = rdd02.map(lambda e: (e, 1))
rdd04 = rdd03.reduceByKey(lambda a,b: a+b)
rdd04.collect()
# 结果如下：
# [(u'', 1), (u'is', 16), (u'ivanl002', 1), (u'ivanl004', 1), (u'\u5f20\u4e39\u5cf0', 1), (u'the', 16), (u'\u563f\u563f\u563f', 1), (u'king', 16), (u'ivanl001', 12), (u'of', 16), (u'ivanl003', 1), (u'\u5389\u5bb3\u554a', 1), (u'world!', 16), (u'ivanl005', 1), (u'h', 1), (u'\u54c8\u54c8\u54c8', 1)]

```