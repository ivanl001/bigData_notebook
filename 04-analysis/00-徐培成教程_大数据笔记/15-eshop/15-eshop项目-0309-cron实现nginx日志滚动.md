* 

### 1, 先写脚本把nginx的日志滚动性的发到flume监听的目录中
```shell
#!/bin/bash

# 时间,到分钟,这个可以根据我们的定时任务来决定，比如说一分钟拷贝一
次，就可以到分钟，一个小时拷贝一次，到小时也是ok的
dataformat=`date +%Y-%m-%d-%H-%M`

# 把原始的日志文件定时拷贝到新文件中
cp /data/nginx/access.log /data/nginx/access_$dataformat.log

# 获取当前的主机名，因为后面flume会同时收集多个服务器文件，所以需要用主机名识别一下
host=`hostname`


# 通过流处理在每行的头部加上服务器名字,流处理器不太懂，后面需要学习一下
sed -i 's/^/'${host}',&/g' /data/nginx/access_$dataformat.log

# 计算拷贝文件的行数，方便从原始的access.log文件中删除
lines=`wc -l < /data/nginx/access_$dataformat.log`

#move access-xxx.log flume's spooldir, 先把拷贝后的文件转移到flume
搜集的文件夹中去，方便flume进行搜集
mv /data/nginx/access_$dataformat.log /data/flume/estore_logs


#delete rows，通过流处理删除已经拷贝的那些行
sed -i '1,'${lines}'d' /data/nginx/access.log


#reboot nginx , otherwise log can not roll.
# nginx杀死后会自动重启
kill -USR1 `cat /var/run/nginx.pid`
# 需要更新一下nginx，不然的话nginx不能进行写入
#nginx -s reload
```

### 2, 执行定时任务crontab
```shell
# 每隔一个小时执行一次
*/60 * * * * prepareLog.sh
```

### 3, 大功告成！