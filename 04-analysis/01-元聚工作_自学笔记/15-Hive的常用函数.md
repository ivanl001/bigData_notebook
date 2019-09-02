### 一，时间函数

#### 1、UNIX时间戳转日期函数: from_unixtime

*这个就是把时间戳转成指定格式的时间字符串*

**语法**: from_unixtime(bigint unixtime[, string format])

**返回值**: string

**说明**: 转化UNIX时间戳（从1970-01-01 00:00:00 UTC到指定时间的秒数）到当前时区的时间格式

> hive> select from_unixtime(1547777456,'yyyyMMdd HH:mm:ss')
>
> 20190118 10:10:56

#### 2、获取当前UNIX时间戳函数: unix_timestamp

*这个就是获取unit时间戳*

**语法**: unix_timestamp()
**返回值**: bigint
**说明**: 获得当前时区的UNIX时间戳

> hive> select unix_timestamp()
>
> 1547777820

#### 3、日期转UNIX时间戳函数: unix_timestamp

*这个就是2那个函数，这个函数还可以把yyyy-MM-dd HH:mm:ss格式(默认，也可以自行指定，看4)的字符串转成unix时间戳*

**语法**: unix_timestamp(string date)
**返回值**: bigint
**说明**: 转换格式为"yyyy-MM-dd HH:mm:ss"的日期到UNIX时间戳。如果转化失败，则返回0。

> hive> select unix_timestamp('2019-01-18 10:10:56') 
>
> 1547777456

#### 4, 指定格式日期转UNIX时间戳函数: unix_timestamp

*指定时间字符串格式，转换成unit时间戳*

**语法**: unix_timestamp(string date, string pattern)
**返回值**: bigint
**说明**: 转换pattern格式的日期到UNIX时间戳。如果转化失败，则返回0。

> hive> select unix_timestamp('2019-01-18 10:10:56','yyyy-MM-dd HH:mm:ss')
>
> 1547777456

#### 5、日期时间转日期函数: to_date

**语法**: to_date(string timestamp)
**返回值**: string
**说明**: 返回日期时间字段中的日期部分。

> hive> select to_date('2019-01-18 10:10:56')
>
> 2019-01-18

#### 6、日期转年函数: year

**语法**: year(string date)
**返回值**: int
**说明**: 返回日期中的年。

> hive> select year('2019-01-18 10:10:56')
>
> 2019

#### 7、日期转月函数: month

**语法**: month (string date)
**返回值**: int
**说明**: 返回日期中的月份。

> hive> select month('2019-01-18 10:10:561')
>
> hive> select month('2019-01-18')
>
> 1

#### 8、日期转天函数: day

**语法**: day (string date)
**返回值**: int
**说明**: 返回日期中的天。

> hive> select day('2019-01-18 10:10:56')
>
> hive> select day('2019-01-18')
>
> 18

#### 9、日期转小时函数: hour

**语法**: hour (string date)
**返回值**: int
**说明**: 返回日期中的小时。

> hive> select hour('2011-12-08 10:03:01') 
> 10

#### 10、日期转分钟函数: minute

**语法**: minute (string date)
**返回值**: int
**说明**: 返回日期中的分钟。

> hive> select minute('2011-12-08 10:03:01')
> 3

#### 11、日期转秒函数: second

**语法**: second (string date)
**返回值**: int
**说明**: 返回日期中的秒。

> hive> select second('2011-12-08 10:03:01')
> 1

#### 12、日期转周函数: weekofyear

**语法**: weekofyear (string date)
**返回值**: int
**说明**: 返回日期在当前的周数。

> hive> select weekofyear('2011-12-08 10:03:01')
> 49

#### 13、日期比较函数: datediff

*返回时间差, 精确到天数，前面减去后面的*

* 时间串的格式这样：yyyy-MM-dd或者yyyy-MM-dd mm:HH:ss

**语法**: datediff(string enddate, string startdate)
**返回值**: int
**说明**: 返回结束日期减去开始日期的天数。

> hive> select datediff('2019-01-18 10:10:56','2018-01-18 10:10:56')
>
> 365

#### 14、日期增加函数: date_add

*返回加上天数的日期，精确到天*

**语法**: date_add(string startdate, int days)
**返回值**: string
**说明**: 返回开始日期startdate增加days天后的日期。 

> hive> select date_add('2019-01-18 10:10:56',1)
>
> 2019-01-19

#### 15、日期减少函数: date_sub

**语法**: date_sub (string startdate, int days)
**返回值**: string
**说明**: 返回开始日期startdate减少days天后的日期。

> hive> select date_sub('2019-01-18 10:10:56', 3)
>
> 2019-01-15

### 二， 条件函数

###### https://www.iteblog.com/archives/2258.html#3UNIX_unix_timestamp