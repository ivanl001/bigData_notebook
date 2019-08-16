

| 目录                                                         | 指向                           |      |
| ------------------------------------------------------------ | ------------------------------ | ---- |
| /etc/hadoop/conf                                             | /etc/alternatives/hadoop-conf  |      |
| /etc/alternatives/hadoop-conf                                | /etc/hadoop/conf.cloudera.yarn |      |
| /opt/cloudera/parcels/CDH-5.7.2-1.cdh5.7.2.p0.18/lib/hadoop/etc/hadoop | /etc/hadoop/conf.cloudera.yarn |      |

![image-20190511225846244](assets/image-20190511225846244.png)



cdh的所有的配置文件在/etc/下面哈， 真正的物理路径而不是逻辑路径

yarn:/etc/hadoop/conf.cloudera.yarn



| 应用      | 配置文件路径                                                 |
| --------- | ------------------------------------------------------------ |
| hdfs/yarn | /etc/hadoop/conf.cloudera.yarn                               |
| hive      | /etc/hive/conf.cloudera.hive                                 |
| hbase     | /etc/hbase/conf.cloudera.hbase                               |
| zookeeper | /opt/cloudera/parcels/CDH-5.7.2-1.cdh5.7.2.p0.18/etc/zookeeper/conf.dist |
| kafka     | /opt/cloudera/parcels/KAFKA-4.0.0-1.4.0.0.p0.1/etc/kafka/conf.dist |
|           |                                                              |
| hue       | /opt/cloudera/parcels/CDH-5.7.2-1.cdh5.7.2.p0.18/etc/hue/conf.empty |
| oozie     | not found                                                    |
| impala    | /opt/cloudera/parcels/CDH-5.7.2-1.cdh5.7.2.p0.18/etc/impala/conf.dist |
|           |                                                              |



