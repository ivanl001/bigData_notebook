```shell
让命令行提示符显式完整路径
---------------------------
1.编辑profile文件，添加环境变量PS1
[/etc/profile]
export PS1='[\u@\h `pwd`]\$'

2.source	
$>source /etc/profile
```

