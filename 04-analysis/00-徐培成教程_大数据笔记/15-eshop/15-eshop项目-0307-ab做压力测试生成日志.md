> ab是apache服务器的一个小组建，所以在mac上的话安装好apache服务器自然就有ab了

* 开启apache服务器的方法
  > sudo apache start
  > sudo apache stop


* 查看ab帮助
  > ab

* 使用ab
*-n代表请求次数，-c代表并发多少* 
  > ab -n 4 -c 2 https://www.baidu.com/

  > localhost好像不行的
  > ab -n 1000 -c 500 http://localhost/zhang.html
  
  > ab -n 4 -c 2 http://127.0.0.1/zhang.html
  

>  apache ab压力测试报错（apr_socket_recv: Connection reset by peer (104)）  

  *查看应用服务器和数据库均未报错，连接被重置，bingyi了以下，apr_socket_recv这个是操作系统内核的一个参数，在高并发的情况下，内核会认为系统受到了SYN flood攻击，会发送cookies（possible SYN flooding on port 80. Sending cookies），这样会减慢影响请求的速度，所以在应用服务武器上设置下这个参数为0禁用系统保护就可以进行大并发测试了：* 
```shell
# vim /etc/sysctl.conf 
net.ipv4.tcp_syncookies = 0
# sysctl -p
```

> 运行ab测试时socket: Too many open files (24)的解决办法

* 所以，基本确定是服务器端打开的文件太多了，超过了限制。google一下：
* 解决办法：
* 查看当前允许打开的文件个数：

* ulimit -a

* 调整可以打开的文件数，一调调到20w：

* ulimit -n 204800