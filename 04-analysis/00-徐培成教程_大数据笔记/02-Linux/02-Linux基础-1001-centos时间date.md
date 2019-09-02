## centos同步时间
* 1.安装ntpdate工具

  > sudo yum -y install ntp ntpdate

* 2.设置系统时间与网络时间同步

  > sudo ntpdate cn.pool.ntp.org

* 3.将系统时间写入硬件时间

  > sudo hwclock --systohc

* 4.查看系统时间
  > timedatectl
  ```
  #得到
        Local time: 四 2017-09-21 13:54:09 CST
    Universal time: 四 2017-09-21 05:54:09 UTC
          RTC time: 四 2017-09-21 13:54:09
         Time zone: Asia/Shanghai (CST, +0800)
       NTP enabled: no
  NTP synchronized: no
   RTC in local TZ: yes
        DST active: n/a
  ```

* 如果没有执行步骤3，则Local time与RTC time显示的值可能不一样

* 5， 更改时区

  > timedatectl set-timezone Asia/Shanghai

```
1、timedatectl查看时间各种状态：
Local time: 四 2014-12-25 10:52:10 CST
Universal time: 四 2014-12-25 02:52:10 UTC
RTC time: 四 2014-12-25 02:52:10
Timezone: Asia/Shanghai (CST, +0800)
NTP enabled: yes
NTP synchronized: yes
RTC in local TZ: no

2、timedatectl list-timezones: 列出所有时区
3、timedatectl set-local-rtc 1 将硬件时钟调整为与本地时钟一致, 0 为设置为 UTC 时间
4、timedatectl set-timezone Asia/Shanghai 设置系统时区为上海
--------------------- 
作者：superlovelei 
来源：CSDN 
原文：https://blog.csdn.net/superlover_/article/details/83655646 
版权声明：本文为博主原创文章，转载请附上博文链接！
```



## centos获取时间
* 1, 获取时间
  *%H是24小时制, %I是12小时制* 
  > date +%Y/%m/%d/%H:%M:%S   
  >
  >   * 2018/12/25/12:22:18

* 2, 时间加减：加一天
  
  > date -d "1 day" +%Y/%m/%d

* 3, 减一天

  > date -d "-1 day" +%Y/%m/%d

* 4, 加一个月

  > date -d "1 month" +%Y/%m/%d

* 5, 减一个月

  > date -d "-1 month" +%Y/%m/%d

* 6, 加一分钟或者减一分钟
  > date -d "+1 minute" +%Y/%m/%d/%H:%M:%S 
  > date -d "-1 minute" +%Y/%m/%d/%H:%M:%S 