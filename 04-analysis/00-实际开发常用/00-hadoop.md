> 实际开发中：
>
> hdfs中使用的文件压缩及文件格式组合比较多的是：
>
> snappy压缩 + orc格式
>
> lzo压缩+orc格式也行
>
> 这个组合能把100G的内容压缩到10G左右



1, 检查本地支持的压缩方式

```shell
hadoop checknative

# 如下是打印结果，说明各种都支持
19/08/06 12:12:37 INFO bzip2.Bzip2Factory: Successfully loaded & initialized native-bzip2 library system-native
19/08/06 12:12:37 INFO zlib.ZlibFactory: Successfully loaded & initialized native-zlib library
Native library checking:
hadoop:  true /opt/cloudera/parcels/CDH-5.12.1-1.cdh5.12.1.p0.3/lib/hadoop/lib/native/libhadoop.so.1.0.0
zlib:    true /lib64/libz.so.1
snappy:  true /opt/cloudera/parcels/CDH-5.12.1-1.cdh5.12.1.p0.3/lib/hadoop/lib/native/libsnappy.so.1
lz4:     true revision:10301
bzip2:   true /lib64/libbz2.so.1
openssl: true /lib64/libcrypto.so

```

