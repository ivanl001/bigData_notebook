1, 分区

2，需要的情况下分桶，本人用的比较少

3，小文件合并，减少mapper数量

4，如果小文件咩有合并，需要操作，开启jvm重用

5，需要连接操作的时候开启连接优化，小表自动加载内存，不然就用连接暗示

6，开启并发执行

7，最优化mapper和reducer个数

8，解决数据倾斜

```sql
SET hive.optimize.skewjoin=true; 
--If there is data skew in join, set it to true. Default is false.

SET hive.skewjoin.key=100000;
--This is the default value. If the number of key is bigger than this, the new keys will send to the other unused reducers.

-- 下面是解决groupby的时候遇到的数据倾斜需要再增加的配置
SET hive.groupby.skewindata=true;

```







