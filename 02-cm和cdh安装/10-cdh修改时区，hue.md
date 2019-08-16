https://blog.csdn.net/yaojianguo1234/article/details/86063021



1. HUE时区
Hue的默认时区为America/Los_Angeles，这里需要修改为Asia/Shanghai
操作：
cm面板 -> HUE -> 配置 -> 搜索[time_zone]， 填写【Asia/Shanghai】

保存 -> 重启hue

2. 修改impala的时区
默认的impala的时区也是有八个小时的时差
操作
cm面板 -> impala -> 配置 -> 搜索[Impala Daemon 命令行参数高级配置代码段（安全阀）]， 填写【use_local_tz_for_unix_timestamp_conversions=true】

保存 -> 重启impala

3. 修改oozie的时区
Oozie默认采用的也是utc时间，那么在很多的调度任务中，会出现很多的时间问题，时间处理这一块也得花费一些时间。
那么接下来就来修改默认时区为中国的时区

登录Cloudera Manager 进入Ooize服务的配置界面搜索“oozie-site.xml”

操作：
cm面板 -> Oozie -> 配置 -> 搜索[oozie-site.xml]， 填写:

<property>
    <name>oozie.processing.timezone</name>
    <value>GMT+0800</value>
</property>
1
2
3
4
保存 -> 重启Oozie

测试
唯一要注意的就是，schedule需要重启才能生效，否则还会以之前的时区在run
--------------------- 
作者：迷失技术de小猪 
来源：CSDN 
原文：https://blog.csdn.net/yaojianguo1234/article/details/86063021 
版权声明：本文为博主原创文章，转载请附上博文链接！