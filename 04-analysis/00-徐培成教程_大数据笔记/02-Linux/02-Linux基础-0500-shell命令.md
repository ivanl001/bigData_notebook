*''在shell中相当于转译，任何中间的符号都不会起作用*

## 1，shell变量

### 1.1，用户自定义变量

*用户自定义的变量，作用域限于当前的shell环境中*

> 定义时赋值变量

```shell
name=ivanl001
```

> 把执行结果赋值给自定义变量

```shell
theList=`ls -la`
# 上面的``也可以用$()代替， $(())在做运算嵌套的时候特别好用，``在做运算嵌套的时候需要转义，相对麻烦，具体看后面的运算符
theList=$(ls -la)
```

```shell
ip=192.168.10.10   //把ip地址赋给一个变量
echo $ip           //通过这个可以取出ip
ping `echo $ip`    //通过这个可以嵌套的ping 192.168.10.10
ping $(echo $ip)   //这样也是可以的，完全没问题
```



> 把一个变量赋值给另外一个变量

```shell
ipaddr=192.168.147.147
# 如何把ipaddr取出来，比如说拼接成其他字符串
url=${ipaddr}:8080
# 或者如下
url="$ipaddr":8080

```

> 删除变量

```shell
unset theList
# 查看变量
set
```



### 1.2，系统变量

*作用域是当前shell环境和其所有子shell环境*

```shell
name=ivanl001
echo name #这里有值
bash # 新建shell子环境
echo name #这里是没有值的，因为自定义的变量作用域限当前shell，子环境中不起作用
exit # 回到name的那个shell环境
export name # 这个可以导出，使其在子环境shell中也可以
bash
echo $name
```



### 1.3，位置参数变量

`$?` :上一个执行命令的执行结果, 0代表成功或者肯定，1代表失败或者否定，这个其实是预定义变量

`$#` :参数的个数

`$1` :第一个参数的内容，取出第几个参数，数字就写几，如果数字大于10， 需要括号，${10}

`$@` :取出所有的参数

`$*` : 也是取出所有的参数，和$@不同的地方是这个会把所有的参数当成一个整体，而\$@则会分别对待每个参数

 



### 1.4，预定义变量

> $? : 上一个执行命令的执行结果, 0代表成功，1代表失败

> $$: 表示当前进程的进程号

> $!:后台运行的最后一个的进程号，也即是最后一个被放入后台运行的进程的pid，后台运行  cmd &



```shell
#!/bin/bash

echo The first shell run successfully!
echo The count of params is: $#; #打印参数的个数
echo The first param is: $1; #打印第一个参数
echo All the params is: $@; #打印所有参数

# 变量的定义与使用
# 1, 环境变量的使用，也即是系统中带有的变量，用$符号
echo The java path is :$JAVA_HOME;

echo -----------------'$*'---------------
for i in $*;
do echo $i;
done

echo ----------------'$@'--------------
for i in $@;
do echo $i;
done

echo ---------------'"$*"'------------
for i in "$*";
do echo $i;
done

echo --------------'"$@"'-------------
for i in "$@";
do echo $i;
done

echo --------------'shift'------------
shift
echo $*

echo -------------'shift loop'---------
for i in $@;
do echo $i;
shift;
done
```

## 2，shell脚本的执行三种方式和read命令

### 2.1,  source或.， 这个是在当前shell环境执行

`source runShell.sh`

`. runShell.sh`

### 2.2, 调用命令执行

`bash runShell.sh`

`sh runShell.sh`

### 2.3, 相对路径和绝对路径执行

`./runShell.sh`

`/home/master/shell/runShell.sh`

* 第一种是在当前的shell环境中执行的，后面两种会新开一个shell


### 2.4,  read命令

```shell
#! /bin/bash

# read命令相当于屏幕输入

read -p "please enter your name:" name name01 name02

echo '你输入的名字是：'$name
echo 'name01:'$name01
echo 'name02'$name02

# 这里是隐式输入，不是显示出来
read -sp "please enter your password:" password
echo '你输入的密码这里不再显示******'
```

## 3，运算符

### 3.1,\`expr 3 + 2\`



### 3.2,\$((3 + 2))



```shell
#! /bin/bash

# 这样是不对的，直接报错命令找不到，可以更改成后面的两种方式
#sum01 = 3 + 2

# 第一种方式，必须要有空格, 如果想要把运算结果赋值，需要家``, 另外需要注意：等号前后都不能有空格
sum01=`expr 3 + 2`
echo 'sum01:'$sum01

# 还是第一种方式 expr 10 + 10
num01=10
num02=20
sum02=`expr $num01 + $num02`
echo 'sum02:'$sum02

# 第二种方式  $(())
sum03=$(($num01 + $num02))
echo 'sum03:'$sum03


echo '---------------下面计算(3+2)*5----------------'
echo '方式一：`expr \`expr 3 + 3\` \* 5`'
sum04=`expr \`expr 3 + 2\` \* 5`
echo 'sum04:' $sum04


echo '方式二：$(((3+2)*5)), 一共三对括号，外面两对是运算，中间一对是先让加号运算'
sum05=$(((3+2)*5))
echo 'sum05:' $sum05
```

## 4, 几个常用符号的比较

### 4.1，``

> *``*  里面的当成命令进行执行

### 4.2, $()

> *$()* 和飘号相同，也是把里面的当成命令进行执行

### 4.3, ${}

> *${}*  取变量值，比如之前的ip拼接的案例中取出ip，url=${ipaddr}:8080，一般情况下用\$ipaddr也可以取出来，但是如果后面要拼接数字活着字母的时候，比如说\$ipaddr8080的时候，这个时候系统就不能判断是要取出ipaddr还是ipaddr8080了， 所以用\${ipaddr}8080轻松解决

### 4.4, $(())

> $(())  用于数值运算等

## 5，条件判断(测试)

> 方式一： test expression

> 方式二：[expression]

*注意：expression前后和 [ ] 都有一个空格*

*eg: [];echo $?*

* 测试范围：字符串，数字，文件
* 表达式结果为真，test返回值为0，用$?接受后即可看到, 否则为非0

### 5.1，字符串测试

`test str01 == str02`: 是否相等，相等为0

`test str01 != str02` : 是否不想等，不想等为0

`test str01` : 是否为不为空，为空返回0

`test -n str01` : 是否不为空，不为空，返回0

`test -z str01` : 是否为空，为空，返回0

```shell
#! /bin/bash

#---------- 下面的分号是命令连接符，连接两个命令

name=ivanl001

# 判断相等
test $name == ivanl001; echo $?

# 判断不想等
test $name != ivanl001; echo $?

# 判断不是空
test $name ; echo $?

# 判断不是空
test -n $name; echo $?

# 判断是空
test -z $name; echo $?

# ----结果是：0 1 0 0 1
```



* 所有的test都可以用[]取代，如下：后面的例子直接用[]

```shell
#! /bin/bash

#---------- 下面的分号是命令连接符，连接两个命令

name=ivanl001

# 判断相等
[ $name == ivanl001 ]; echo $?

# 判断不想等
[ $name != ivanl001 ]; echo $?

# 判断不是空
[ $name ]; echo $?

# 判断不是空
[ -n $name ]; echo $?

# 判断是空
[ -z $name ]; echo $?

# ----结果是：0 1 0 0 1
```



### 5.2，数字测试
> 注意：空格一定要空！！！空格一定要空！！！！空格一定要空！！！！！

`[ 3 -eq 4 ]`: 判断相等

`[ 3 -ge 4 ]`: 判断大于或者等于

`[ 3 -gt 4 ]`: 判断大于

`[ 3 -le 4 ]`： 判断小于等于

`[ 3 -lt 4 ]`：判断小于

`[ 3 -ne 4 ]`: 判断不相等

```shell
#! /bin/bash

num01=4
num02=5

# 判断等于
[ $num01 -eq $num02 ]; echo $?

# 判断不等于
[ $num01 -ne $num02 ]; echo $?

# 判断大于等于
[ $num01 -ge $num02 ]; echo $?

# 判断大于
[ $num01 -gt $num02 ]; echo $?

# 判断小于等于
[ $num01 -le $num02 ]; echo $?

# 判断小于
[ $num01 -lt $num02 ]; echo $?

# 结果是：1 0 1 1 0 0
```





### 5.3，文件测试

`[ -d file ]`:判断是否目录 

`[ -e file ]`: 判断是否存在

`[ -f file ]`: 判断是否是常规文件

`[ -L file ]`: 判断是否是连接

`[ -r file ]`: 判断是否可读

`[ -w file ]`: 判断是否可写

`[ -x file]`: 判断是否可执行



```shell
#! /bin/bash

# 文件系统的判断

file=log.txt

# 是否是文件夹
[ -d $file ]; echo $?
# 是否存在
[ -e $file ]; echo $?
# 是否文件
[ -f $file ]; echo $?
# 是否可读
[ -r $file ]; echo $?
# 是否可写
[ -w $file ]; echo $?
# 是否可执行
[ -x $file ]; echo $?

# 结果是： 1 0 0 0 0 1
```



### 5.4, 多重条件

`[ A -a B ]`: 与

`[ A -o B]`: 或

`[ !A ]` : 非,也就是取反

```shell
# /bin/bash

str01=ivanl001
num=10

# 判断str01等于ivanl001而且num的值大于5
[ $str01 == ivanl001 -a $num -gt 5 ]; echo $?

# 判断str01等于ivanl002或者num的值大于20
[ $str01 == ivanl002 -a $num -gt 20 ]; echo $?

# 判断非，也即是取反
[ ! $str01 == ivanl001 ]; echo $?

# 在这里加上&&和||命令
[ ! $str01 == ivanl001 ] && echo success || echo failed ; echo ok,all done!

# 结果是： 0 1 1 failed ok,all done!
```



> 有一点需要注意的是：上面的是判断条件， 而&&或者||是命令

`bash log.txt && echo success || echo failed ; echo program finished`

## 6, 流程控制语句

### 6.1, if语句

```shell
# /bin/bash

# 单分支if语句
read -p "Please input your hostname:" hostname

if [ $hostname != root ]; then echo 'Sorry, you do not have permission to opearte this;'; else echo 'ok, you can go in now!';fi

num=1
if [ $num -lt 10 ];
then echo 'current num is:' $num
# 注意这里是赋值，赋值的时候等号左右两边不能有空格
num=`$num + 1`
fi

read -p "Please input your age: " age
if [ $age -lt 20 ]; then echo 'you are too young';
elif [ $age -lt 40 ]; then echo 'ok, perfect';
else echo 'you are too old!';
fi
```



### 6.2，case语句

```shell
#/bin/bash

# case语句的使用
case $1 in
start)
        echo 'starting!'
        service httpd start;
        ;;
stop)
        echo 'stopping'
        service httpd stop;
        ;;
*)
        echo 'usage:{start|stop}';
esac
```



### 6.3，for循环

> for循环可以循环数字，字符数组，执行结果等，具体看下面代码

```shell
# --------------数字型循环-------------
# 第一种for循环写法
for((i=0;i<10;i++));
do echo $i;
done

# 第二种for循环,记住要有$
for i in $(seq 1 10);
do echo '第二种：' $i
done

# 第三种for循环, 这里是中括号
for i in {1..10};
do echo '第三种:' $i
done

# 第四种for循环,这个不太会，后面看
awk 'BEGIN{for(i=1; i<=10; i++) print i}'


# -----------------字符型循环----------------
# 第一种字符循环:循环执行命令后的结果
for i in `ls`;
do echo $i is the file name;
done

# 第二种循环输入的参数
for i in $*;
do echo $i is the input char;
done

# 第三种循环给定的多个参数
f1=ivanl001
f2=ivanl002
f3=ivanl003
for i in f1 f2 f3;
do echo $i is the param;
done

# 第四种循环数组中的参数
list="ivanl001 is the king of world!"
for i in $list;
do echo $i is the one of list;
done


# ---------------路径查找-----------------
# 第一种路径循环
for file in /home/master/*
do echo $file is the file of master;
done

# 第二种路径循环
for file in $(ls *.sh);
do echo $file is the sh file;
done
```



### 6.4,  while循环

```shell
#/bin/bash

# while循环求100内数字的和
i=0
sum=0
while(($i<=100));
do sum=$(($sum+$i));i=$(($i+1));
done

echo 'sum='$sum
```

## 7, 自定义函数和脚本调试

### 7.1，自定义函数

```shell
#/bin/bash

# 自定义函数
echo '自定义函数'

function  start(){
  echo 'starting...'
  echo 'the param is:'$1
  return 100
}

function stop(){
  echo 'stopping...'
  return 101
}

function restart(){
  echo 'restarting...'
  return 102
}

start $1
```



### 7.2，脚本调试

> bash -n errorTest.sh # 这里是不运行，直接调试代码语法

> bash -x errorTest01.sh #这个是边运行，边打印你的代码

```shell
[master@master shell]$ bash -x errorTest01.sh
+ read -p 'Please input your hostname:' hostname
Please input your hostname:zang
+ '[' zang '!=' root ']'
+ echo 'Sorry, you do not have permission to opearte this;'
Sorry, you do not have permission to opearte this;
+ num=1
+ '[' 1 -lt 10 ']'
+ echo 'current num is:' 1
current num is: 1
+ num=2
+ read -p 'Please input your age: ' age
Please input your age:
```

> bash -v errorTest01.sh # 这里会打印出所有代码，包括判断语句中不执行的和注视

```shell
[master@master shell]$ bash -v errorTest01.sh
# /bin/bash


# 单分支if语句
read -p "Please input your hostname:" hostname
Please input your hostname:zhang

if [ $hostname != root ]; then echo 'Sorry, you do not have permission to opearte this;'; else echo 'ok, you can go in now!';fi
Sorry, you do not have permission to opearte this;


# 下面需要注意的是：赋值的时候等号两边不能有空格
num=1
if [ $num -lt 10 ];
then echo 'current num is:' $num
num=$(($num + 1))
fi
current num is: 1



read -p "Please input your age: " age
Please input your age: 99
if [ $age -lt 20 ]; then echo 'you are too young';
elif [ $age -lt 40 ]; then echo 'ok, perfect';
else echo 'you are too old!';
fi
you are too old!
```

>  set -x errorTest01.sh #这个是只对部分代码进行调试，也即是代码中有set -x字样的那些代码

## 8，定时器

## 100，后台运行

> 第一种方式：&，就是在命令后面直接 &就可以了

`./shell02.sh >> log.txt &`

> 第二种方式， nohup

`nohup ./shell02.sh >> log.txt`: 这个会占用命令行

`nohup ./shell02.sh >> log.txt &`: 这个不占用命令行，但问题是我直接用第一种方式不就行了，干嘛用这个？？？

## 执行命令的操作

`bash log.txt && echo success || echo failed ; echo program finished`

> 上面的命令， bash可以让那些没有执行权限的文件执行（指的是当前用户，用当前用户执行其他用户的文件，不代表一定可以执行成功），&&代表前面的命令执行成功后才会执行，||代表执行失败才会执行，;代表如论如何都会执行，这个好像跟try-catch-finally有点像，&&相当于try，|| 相当于catch，；相当于finally