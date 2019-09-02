*大概的意思是：创建hive表，然后把清洗后符合要求的数据放到指定的目录下，这个目录的规则需要符合分区的要求，然后动态的通过shell添加分区，自动的读取hdfs上的内容*

## hive数据库相关操作

* 1, 创建数据库
  > create database estore;

* 2, 创建表
  > create external table estore.logs01 (
hostname string,
remote_addr string,
remote_user string,
time_local string,
request string,
status string,
body_bytes_sent string,
http_referer string,
http_user_agent string,
http_x_forwarded_for string
)
partitioned by(year int ,month int,day int,hour int,minute int)
row format DELIMITED 
FIELDS TERMINATED BY ',' 
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;

* 3, 添加分区
  > alter table estore.logs01 add partition (year=2018, month=12, day=25, hour=17, minute=56);


！！！！！！！有一点需要值得特别特别特别注意：添加分区的时候注意文件夹是没有0的，year=2018/month=12/day=25/hour=8/minute=12，这里只能是8，不能是08，如果想要直接存到对应的目录里，注意：一定不要存入到08目录下面了，要不然有的折腾！！！！！ 

## 使用脚本添加hive分区

* 1, 基础命令,这个是shell命令，不需要启动hive，在centos上可以直接运行
  > hive -e "alter table estore.logs add if not exists partition (year=2018, month=02, day=01, hour=13, minute=13) partition (year=2018, month=03, day=01, hour=13, minute=13)"

* 2, 定时任务(shell脚本内容)
*hive_addPartition.sh, 加入到定时任务中，可以定时创建分区*
  ```shell
  #!/bin/bash
  
  # date -d "+1 minute" +%Y/%m/%d/%H:%M:%S
  year=`date +%Y`
  month=`date +%m`
  day=`date +%d`
  hour=`date +%H`
  minute=`date +%M`
  before_minute=`date -d "-1 minute" +%M`
  after_minute=`date -d "+1 minute" +%M`
  
  echo $year-$month-$day $hour:$minute
  echo '之前一分钟:'$before_minute
  echo '之后一分钟:'$after_minute
  
  hive -e "alter table estore.logs add if not exists partition (year=${year}, month=${month}, day=${day}, hour=${hour}, minute=${minute}) partition (year=${year}, month=${month}, day=${day}, hour=${hour}, minute=${before_minute}) partition (year=${year}, month=${month}, day=${day}, hour=${hour}, minute=${after_minute})"
  ```
!!!!!!!注意：crontab -e中添加hive -e这样的脚本的时候，crontab的环境变量是有问题的，需要 把环境变量写入到脚本中才行哈：按照如下方式进行处理:
```shell
# 这个是crontab的脚本，也就是/var/spool/cron/root中的内容
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/hive/bin:/usr/local/hadoop/bin:/usr/local/hadoop/sbin:/usr/local/hbase/bin

* * * * * echo $PATH >> /data/ivanl001.txt
* * * * * prepareLog.sh
* * * * * hive_addPartition.sh
```

load data inpath '/user/estore/cleaned/year=2018/month=12/day=25/hour=18/minute=11/master_done.log' into table logs01 partition (year=2018, month=12, day=25, hour=18, minute=11)



