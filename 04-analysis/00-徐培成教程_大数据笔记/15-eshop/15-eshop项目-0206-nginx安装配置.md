## 一，centos上ngix安装

* 1、添加源

  *默认情况Centos7中无Nginx的源，最近发现Nginx官网提供了Centos的源地址。因此可以如下执行命令添加源：*

  > sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm


* 2、安装Nginx

  *通过yum search nginx看看是否已经添加源成功。如果成功则执行下列命令安装Nginx。*

  > sudo yum install -y nginx


* 3、启动Nginx并设置开机自动运行

  > sudo systemctl start nginx.service
  > sudo systemctl enable nginx.service


* 4、浏览查看效果

  > http://master:80
  
* 5, 安装位置
  > whereis nginx

  * nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz
  
  ```java
  以下是Nginx的默认路径： 
  * /var/run/nginx.pid这个可以找到nginx的进程号
  (1) Nginx配置路径：/etc/nginx/ 
  (2) PID目录：/var/run/nginx.pid 
  (3) 错误日志：/var/log/nginx/error.log 
  (4) 访问日志：/var/log/nginx/access.log 
  (5) 默认站点目录：/usr/share/nginx/html
  事实上，只需知道Nginx配置路径，其他路径均可在/etc/nginx/nginx.conf 以及/etc/nginx/conf.d/default.conf 中查询到。
  ```
  

## 二，mac上nginx安装

* 1, brew 搜索软件
  
> brew search nginx
  
* 2, brew 安装软件
  
> brew install nginx
  
* 3, brew 卸载软件
  
> brew uninstall nginx
  
* 4, brew 升级
  
> sudo brew update
  
* 5, 查看安装信息(经常用到, 比如查看安装目录等)
  
> sudo brew info nginx
  
* 6, 查看已经安装的软件
  
> brew list
  
* 启动nginx
  
> sudo brew services start nginx
  
* 浏览器查看效果
  
> http://localhost:8080
  
* nginx的安装位置:
  > nginx -V
  * 查找nginx各种位置
  * /usr/local/etc/nginx

  > where nginx
  * /usr/local/bin/nginx
  * /usr/local/Cellar/nginx/1.15.3//bin/nginx

## 三，centos和mac上查看端口占用情况：

* centos：
  *a代表all,不带a默认是estabed的那些连接，n代表把显示ip而不是hostname, o代表display timers，p代表显示相关的进程*
  
> netstat -anop | grep 80
  
* mac
  *-i代表selects the listing of files any of whose Internet address matches the address specified in i，这里可以匹配pid，也可以匹配端口，如下*
  > lsof  -i:8080 

  > lsof -i | grep pid
  > lsof  -i:port

## 四，nginx配置

## 五，防盗链
*这个之前写爬虫的时候遇到过，如果访问一个页面中的超链接的时候，进去的时候需要把当前的地址一并带过去，验证通过才允许访问，直接拷贝链接进行访问是被阻止的，其实也就是设置了http_referer的缘故*