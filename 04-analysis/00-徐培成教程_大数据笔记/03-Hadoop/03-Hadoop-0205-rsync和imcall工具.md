## 1，rsync

### 01，本地同步文件夹

> rsync -av ivanl001/ test/ #test文件夹不存在的时候会自动创建

### 02，本地同步到远程服务器上

> rsync -avl ivanl001/ root@slave01:/home/test # l代表link，链接

```shell
#! /bin/bash

# 这个脚本的作用是同步文件或者文件夹到远程的三台从服务器上

if(($#!=1)); then echo '请输入正确的需要同步的文件夹或文件的路径';exit;fi;

echo;

path=$@;

relativeDirPath=`dirname $path`;#这里取出来有时候是.,也就是当前目录
cd $relativeDirPath;
dirPath=`pwd`;

basePath=`basename $path`;

fullPath=$dirPath/$basePath;

echo '--------------------------------同步源目录开始:'$fullPath'------------------------------------';

for((i=1;i<4;i++));
do
rsync -avl $@ root@slave0$i:$fullPath;
done;

echo '--------------------------------同步源目录结束:'$dirPath/$basePath'------------------------------------';

echo;

```

* 做参考
```shell
#!/bin/bash

if [[ $# -lt 1 ]] ; then echo no params ; exit ; fi

p=$1
#echo p=$p
dir=`dirname $p`
#echo dir=$dir
filename=`basename $p`
#echo filename=$filename
cd $dir
fullpath=`pwd -P .`
#echo fullpath=$fullpath

user=`whoami`
for (( i = 202 ; i <= 204 ; i = $i + 1 )) ; do
   echo ======= s$i =======
   rsync -lr $p ${user}@s$i:$fullpath
done ;
```

## 2, 通过ssh远程同时操作多台从服务器

```shell
#! /bin/bash

#if((n!=1)); then echo '请输入正确命令';exit;fi;#这里不能判断一个参数，不然ls -a这种就会有问题

echo;#打印一行空格

echo '---------------------------------执行命令开始:'$@'-----------------------------------';

echo;#打印一行空格

echo '---------------master---------------'
ssh localhost "$@";

for((i=1;i<4;i++))
do
echo '---------------'slave0$i'--------------';
ssh slave0$i "$@";
done;

echo;

echo '---------------------------------执行命令结束:'$@'-----------------------------------';
```



