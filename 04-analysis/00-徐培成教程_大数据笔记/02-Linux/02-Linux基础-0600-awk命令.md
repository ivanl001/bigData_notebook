* jps是这种

  ```
  34800 RunJar
  4340 ResourceManager
  8100 RunJar
  4246 DFSZKFailoverController
  70411 HMaster
  6491 Application
  72554 Jps
  69341 Main
  3934 NameNode
  69150 KafkaToHbaseApp
  ```


* 管道给awk(默认空格分隔，打印第一个元素)
  > jps | awk '{print $1}'
  ```
  34800
  4340
  8100
  4246
  72567
  70411
  6491
  69341
  3934
  69150
  ```

* 也可以指定分隔符
  
> less /etc/hosts | awk '{print \$1}' | awk -F '.' '{print $4}'
  
* 动态提取ip地址
  
  > ifconfig | grep inet | head -1 | awk '{print $2}'
 * 上面分别是ifconfig，打印出ip各种信息
 * grep inet， 过滤出inet字样的行
 * head -1， 取出过滤后的第一行
 * awk '{print $2}', 第一行按照空格分隔后取出第二个分隔，也就是ip地址了



```shell
# 我看到Main是第一个，那么就head取一个就好， awk取第一列，也就是Main的Pid
jps | grep "Main" | head -n 1 |  awk '{print $1}'
```



```shell
# 比如说我的hosts文件是这样的
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.147.101 centos01
192.168.147.102 centos02
192.168.147.103 centos03                 
```



```shell
# 然后我想取出ip的最后三个数字， 如下：
less /etc/hosts | tail -3 | awk '{print $1}' | awk -F '.' '{print $4}'

# 打印结果：
101
102
103
```







