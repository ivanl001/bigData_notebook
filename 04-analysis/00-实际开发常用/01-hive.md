## 1, hive执行MR合并文件设置

```mysql
// 输出合并小文件
SET hive.merge.mapfiles = true; -- 默认true，在map-only任务结束时合并小文件
SET hive.merge.mapredfiles = true; -- 默认false，在map-reduce任务结束时合并小文件
SET hive.merge.size.per.task = 268435456; -- 默认256M
SET hive.merge.smallfiles.avgsize = 16777216; -- 当输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件merge
```



## 2, hive执行元聚设置

```mysql
SET hive.exec.dynamic.partition.mode=nonstrict;
SET hive.exec.max.dynamic.partitions=100000;
SET hive.exec.max.dynamic.partitions.pernode=100000;
SET hive.exec.max.created.files=655350;
SET hive.merge.mapredfiles=TRUE;
# 这个视情况使用，大多时候不设置
SET mapreduce.job.reduces=20;
```





