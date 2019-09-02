> 12, 查询近3个月的通话数据

* 查询字符串的截串

* // mysql， 从1开始哈
  > select substring(name,1,4) from test.stu;   
* //hive
  > select substring(caller, 1, 4) from imdb.calllogs_hive_hbase; 

## 12, 查询

* 01, 首先是通过截串来获取月份相关信息

  > select * from imdb.calllogs_hive_hbase where caller = '15338595369' and substring(time, 1, 6) = '201805';

  * 这个会转mr
  > select count(*) , substring(time, 1, 6) from imdb.calllogs_hive_hbase where caller = '15338595369' group by substring(time, 1, 6);

  * 下面这句不好使，是那个大于的问题
  > select count(*) , substring(time, 1, 6) from imdb.calllogs_hive_hbase where caller = '15338595369' and substring(time, 1, 6) > 201805 group by substring(time, 1, 6);
  
  > select count(*) , substring(time, 1, 6) from imdb.calllogs_hive_hbase where caller = '15338595369' and substring(time, 1, 4) = '2018' group by substring(time, 1, 6);
  
  > select count(*) , substring(time, 1, 6) as t from imdb.calllogs_hive_hbase where caller='15732648446' and substring(time, 1, 4)='2017' group by substring(time, 1, 6)

* 02, 查询所有用户各个月份的通话次数
  * 只要是聚合都会转成MR哈
  > select caller, substring(time, 1, 6), count(*) from imdb.calllogs_hive_hbase group by caller, substring(time, 1, 6);
  