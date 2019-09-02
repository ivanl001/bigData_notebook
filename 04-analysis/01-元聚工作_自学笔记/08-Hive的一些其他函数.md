

*5c6a62a5是16进制，conv是进制转换函数，从16进制转成10进制，然后这是个时间戳，把这个时间戳字符串通过cast改变成int类型，from_unixtime函数变成指定时间格式即可*	

* select cast(conv(substring('5c6a62a5',0,8),16,10) as bigint)

* select from_unixtime(cast(conv(substring('5c6a62a5',0,8),16,10) as bigint),'yyyy-MM-dd HH:mm:ss')

