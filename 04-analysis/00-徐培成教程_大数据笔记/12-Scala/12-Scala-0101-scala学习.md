# Scala基础

> Scala其实就是java的脚本化

> scala中不具体分基本类型和引用类型，所有的类型都是类：Byte, Char, Short, Int, Long, Float, Double 7个和Boolean类型

进入scala命令行
  > scala

一段代码在命令行中：
  > 输入:paste后可段落写入，直到按ctrl + d结束


## 1，var和val

 > var是变量，赋值后可以更改

 > val是value, 是常量，赋值后不可更改

 ```scala
  
 var a = "ivanl001";
 var b:String = "ivanl002";
 
 //兼容java类型
 var c:Integer = 10;
 var d = 30;
 //scala的整型
 var e:Int = 40;
 ```


## 2，常用数据类型

* scala:Byte, Char, Short, Int, Long, Float, Double 7个和Boolean类型

* java:

```

byte：Java中最小的数据类型，在内存中占8位(bit)，即1个字节，取值范围-128~127，默认值0

short：短整型，在内存中占16位，即2个字节，取值范围-32768~32717，默认值0

int：整型，用于存储整数，在内在中占32位，即4个字节，取值范围-2147483648~2147483647，默认值0

long：长整型，在内存中占64位，即8个字节-2^63~2^63-1，默认值0L

float：浮点型，在内存中占32位，即4个字节，用于存储带小数点的数字（与double的区别在于float类型有效小数点只有6~7位），默认值0

double：双精度浮点型，用于存储带有小数点的数字，在内存中占64位，即8个字节，默认值0

char：字符型，用于存储单个字符，占16位，即2个字节，取值范围0~65535，默认值为空

boolean：布尔类型，占1个字节，用于判断真或假（仅有两个值，即true、false），默认值false

```

## 3，操作符重载

* 1+2    //运算符其实都是方法，可以按照下面这行调方法的方式进行调用
* 1.+(2)
* 1.toString      //无参数式方法调用
* 1.toString()    //方法调用
* 1 toString      //运算符方式调用
* name(0)//这是取出name中的第一个字符，其实等价于：name.apply(0)

* 在java中函数和方法没有明确的定义，可以理解为函数也就是方法.

> 但是在scala中， 函数是可以直接调用的，不需要依赖对象而存在。
> 而方法是对象的成分， 要调用方法，需要通过对象调用

##### REPL Read, Evalute, Print, Loop,

##### printf("ivanl %s is the king of world!", "ivanl001");

##### 导包方式 import scala.math._  //这里_代表通配符


## 4，条件表达式

> var b = if (a > 1) 3 else 4;

* 下面这种返回值则是Any，相当于java中的Object类型
> var b = if (a > 1) "zhang" else 4;


## 5，类型转换 .to***

> 1.toString

> 1.toString()

> "123".toInt

> "123".toInt()

> "123".toFloat

> "123".toFloat

## 6.1, Unit类型

* var x: Unit = (), 这个相当于java中的void类型


## 6.2，打印

c语言风格
* printf("ivanl010 is the %s", "king");    ===》 ivanl010 is the king

java语言风格
* println("ivanl001" + "iii");


## 7, for循环和yield的用法

*yield可以产生新的集合进行返回*

> for(i<- 0 to 4) println(i);

> for(i <- 0 to 4; j <- 0 to 4) { printf("i=%d, j=%d, i*j=%d", i, j, i*j); println()};

想要前包后不包，可以用until

> for(i <- 0 until 10) println(i);  //这行只会打印到数字9， <- 这个符号会打印最后一个数字10的哈

```
i=0, j=0, i*j=0
i=0, j=1, i*j=0
i=0, j=2, i*j=0
i=0, j=3, i*j=0
i=0, j=4, i*j=0
i=1, j=0, i*j=0
i=1, j=1, i*j=1
i=1, j=2, i*j=2
i=1, j=3, i*j=3
i=1, j=4, i*j=4
i=2, j=0, i*j=0
i=2, j=1, i*j=2
i=2, j=2, i*j=4
i=2, j=3, i*j=6
i=2, j=4, i*j=8
i=3, j=0, i*j=0
i=3, j=1, i*j=3
i=3, j=2, i*j=6
i=3, j=3, i*j=9
i=3, j=4, i*j=12
i=4, j=0, i*j=0
i=4, j=1, i*j=4
i=4, j=2, i*j=8
i=4, j=3, i*j=12
i=4, j=4, i*j=16
```

> for(i <- 0 to 4; j <- 0 to 4 if i!=j) { printf("i=%d, j=%d, i*j=%d", i, j, i*j); println()};

```
i=0, j=1, i*j=0
i=0, j=2, i*j=0
i=0, j=3, i*j=0
i=0, j=4, i*j=0
i=1, j=0, i*j=0
i=1, j=2, i*j=2
i=1, j=3, i*j=3
i=1, j=4, i*j=4
i=2, j=0, i*j=0
i=2, j=1, i*j=2
i=2, j=3, i*j=6
i=2, j=4, i*j=8
i=3, j=0, i*j=0
i=3, j=1, i*j=3
i=3, j=2, i*j=6
i=3, j=4, i*j=12
i=4, j=0, i*j=0
i=4, j=1, i*j=4
i=4, j=2, i*j=8
i=4, j=3, i*j=12
```

> for(i <- 0 to 10) yield i%3;
```scala
res35: scala.collection.immutable.IndexedSeq[Int] = Vector(0, 1, 2, 0, 1, 2, 0, 1, 2, 0, 1)
```

## 8,函数相关

> def add(a:Int, b:Int):Int = a+b;

> def add(a:Int, b:Int):Int = {var c = a+b; print("ivanl001 is the king of world!"); return c;}

*通过函数实现递归阶乘*
> def factorial(n:Int): Int = if (n==1) 1 else n*factorial(n-1);


* 默认参数

> def add(a:Int=10, b:Int=20):Int = a+b;

* 不定数量参数
```scala
def add(params:Int*):Int = {
  var sum = 0;
  for(i <- params){
    sum += i;
  }
  return sum;
}
```

* 调用不定参数的函数时候，如何传入的是范围，可以如下调用：

 `add(0 to 100:_*)`  //这里的意思是：0 to 100这是一个Range对象，和Int类型参数不匹配，所以可以通过:_*把range类型当作序列处理

 * 同时还可以通过递归的函数求和：
   * .head是默认的方法，取出序列的第一个

 ```scala
 def recursiveSum(args:Int*):Int = {
   if(args.length == 0) 0
   else
   args.head + recursiveSum(args.tail: _*)
 }
 ```

  `recursiveSum(0 to 100:_*)`