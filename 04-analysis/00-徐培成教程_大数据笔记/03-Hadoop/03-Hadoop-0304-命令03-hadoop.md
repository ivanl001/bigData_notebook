# 3, hadoop命令



## 3.0, fs: 文件系统命令

```shell
hadoop fs -ls /
# 这个和hdfs dfs -ls /完全等价
```





## 3.1, checknative：检查本地压缩库

```shell
# 检查本地压缩库
hadoop checknative

hadoop:  true /opt/cloudera/parcels/CDH-5.12.1-1.cdh5.12.1.p0.3/lib/hadoop/lib/native/libhadoop.so.1.0.0
zlib:    true /lib64/libz.so.1
snappy:  true /opt/cloudera/parcels/CDH-5.12.1-1.cdh5.12.1.p0.3/lib/hadoop/lib/native/libsnappy.so.1
lz4:     true revision:10301
bzip2:   true /lib64/libbz2.so.1
openssl: false Cannot load libcrypto.so (libcrypto.so: cannot open shared object file: No such file or directory)!
```



## 3.2, format：格式化命令

```shell
hadoop namenode -format
```



## 3.3, classpath：打印类路径

```shell
hadoop classpath
```



