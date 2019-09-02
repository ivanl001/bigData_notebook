> 在解析埋点数据时会遇到两种不同的日期格式：yyyymmdd和yyyy-mm-dd，此类型之间的转换主要有两种思路：

##### 01, 第一种方法：from_unixtime+unix_timestamp

```mysql
-- 20180905转成2018-09-05
select from_unixtime(unix_timestamp('20180905','yyyymmdd'),'yyyy-mm-dd')
--结果如下：
2018-09-05

-- 2018-09-05转成20180905
select from_unixtime(unix_timestamp('2018-09-05','yyyy-mm-dd'),'yyyymmdd')
--结果如下：
20180905

```



##### 02, 第二种方法：substr + concat

```mysql
-- 20180905转成2018-09-05 
select concat(substr('20180905',1,4),'-',substr('20180905',5,2),'-',substr('20180905',7,2))
结果如下：
2018-09-05 

-- 2018-09-05转成20180905
select concat(substr('2018-09-05',1,4),substr('2018-09-05',6,2),substr('2018-09-05',9,2))
结果如下：
20180905
```



##### 下面主要讲解from_unixtime和unix_timestamp两种函数：

###### 01, from_unixtime：时间戳转日期函数

* 用法：from_unixtime(bigint unixtime[, stringformat])
* 返回值: string
* 说明: 转化时间戳到当前时区的时间格式

```mysql
select from_unixtime(1423306743,'yyyyMMdd') 
结果如下：
20150207
```

###### 02, unix_timestamp:日期转时间戳函数

* 用法:unix_timestamp(string date)
* 返回值:   bigint
* 说明: 转换格式为“yyyy-MM-dd HH:mm:ss“的日期到UNIX时间戳。如果转化失败，则返回0。

```mysql
select unix_timestamp('2018-09-05 12:01:03') 
结果如下：
1536120063

-- 获取当前日期的时间戳：
select unix_timestamp() 
结果如下：
1536126324
```

###### 03, 日期相互转换并相减

```sql
-- 日期减去5天
select date_sub('2019-03-07',5)

-- 日期格式转换
select from_unixtime(unix_timestamp('20190307','yyyymmdd'),'yyyy-mm-dd')

-- 20190307格式转成2019-03-07格式，进行日期减去4
select date_sub(from_unixtime(unix_timestamp('20190307','yyyymmdd'),'yyyy-mm-dd'), 4)

-- 20190307格式转成2019-03-07格式，进行日期减去4，然后得到2019-03-03格式，再转回20190303格式
select from_unixtime(unix_timestamp(date_sub(from_unixtime(unix_timestamp('20190307','yyyymmdd'),'yyyy-mm-dd'), 4), 'yyyy-mm-dd'), 'yyyymmdd')

-- 20190307格式转成2019-03-07格式，进行日期减去4，然后得到2019-03-03格式，再转回20190303格式，只不过参数通过accounting_day传入
select from_unixtime(unix_timestamp(date_sub(from_unixtime(unix_timestamp('${accountign_day}','yyyymmdd'),'yyyy-mm-dd'), 4), 'yyyy-mm-dd'), 'yyyymmdd')
```

