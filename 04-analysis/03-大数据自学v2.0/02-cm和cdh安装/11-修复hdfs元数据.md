* 启动hdfs的时候，namenode报错如下：无法启动

```java
java.io.IOException: There appears to be a gap in the edit log. We expected txid ***, but got txid
```



网上查阅资料，说是元数据损坏，需要修复元数据

```shell
hadoop namenode –recover

选择Y

选择c
```



重启namenode节点