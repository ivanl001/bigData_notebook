# Scala基础01


## lazy的用法

> 1, 比如说从文件中读取内容，会直接把文件内容读到变量中去

```scala
 val textStr = scala.io.Source.fromFile("/Users/ivanl001/Desktop/test.txt").mkString;
```

> 2, 如果说不需要立即读取，而是有需求的当用到的时候再进行读取进去，那么就可以用lazy,还有这里只能用val，var是不能用lazy修饰的，因为可变

```scala
lazy val textStr = scala.io.Source.fromFile("/Users/ivanl001/Desktop/test.txt").mkString;

textStr: String = <lazy>
```


## 异常相关处理等

> 和java其实是类似的

```scala

import java.io.IOException

try{
  val textStr = scala.io.Source.fromFile("/Users/ivanl001/Desktop000aa/test.txt").mkString;
  print(textStr);
}catch{
  case ex: IOException =>  print(ex)
  case _:Exception =>  print("catch exception")
}

```

> 1 to 10 :_* //这个的意思是把1 到 10转成序列

## 数组相关

### 创建定长数组

```scala
val nums = new Array[Int](10)
//10个整数的数组， 所有元素的初始化是0

val a = new Array[String](10)
//10个字符串的数组， 所有元素的初始化是null

val s = Array("Hello", "World")
// 已经提供初始值的化就不需要new
```

### 操作定长数组元素，包括赋值和访问

```
//设置元素
s(0) = "Goodbye"

//取出某个元素
s(0)
```


### 创建变长数组
```scala
import scala.collection.mutable.ArrayBuffer

val b = ArrayBuffer[Int]()

//在尾部添加元素
b += 1
// ArrayBuffer(1)

//在尾部添加多个元素
b += (1, 2, 3, 5)
// ArrayBuffer(1, 1, 2, 3, 5)

//++=追加集合
b ++= Array(8, 13, 21)
// ArrayBuffer(1, 1, 2, 3, 5, 8, 13, 21)

//移除元素
b.trimEnd(5)
// ArrayBuffer(1, 1, 2)

//移除头部一个元素
//b.trimStart(1)


b.insert(2, 6)
// ArrayBuffer(1, 1, 6, 2)

b.insert(2, 7, 8, 9)
// ArrayBuffer(1, 1, 7, 8, 9, 6, 2)

b.remove(2)
// ArrayBuffer(1, 1, 8, 9, 6, 2)

b.remove(2, 3)
// ArrayBuffer(1, 1, 2)

val c = b.toArray
//c就是定长数组

for (i <- 0 until a.length)
        println(i + ": " + a(i))

//遍历定长数组
for(i <- 0 until c.length) println(c(i))

//遍历可变数组
 for(i <- 0 until b.length) println(b(i))

```

### 数组转换

```Scala
val a = Array(2, 3, 5, 7, 11);
val result = for (elem <- a) yield 2 * elem;
// result Array(4, 6, 10, 14, 22);

//如果想要丢掉奇树位置的数字：
var result01 = for (elem <- a if elem % 2 == 0) yield 2 * elem;

//创建一个数组
val a = Array(0 to 10:_*)
//过滤后重组然后生成新的数组
val b = a.filter(_ % 2 == 0).map(_ * 2)
//Array[Int] = Array(0, 4, 8, 12, 16, 20)

//第一种方式获取偶数数组，而且还可以运算后返回
val a = for (i <- 0 until 10 if i%2 == 0) yield i;
val a = for (i <- 0 until 10 if i%2 == 0) yield i*2;

//第二种方式
val b = Array(1 to 10 :_*);
//b: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
b.filter(_%2 == 0).map(_ * 2);
//res15: Array[Int] = Array(4, 8, 12, 16, 20)
```

### 数组的其他方法
```scala
//求和
Array(9,8,7).sum

Array(9,8,7).max

Array(9,8,7).min

Array("Mary", "had", "a", "little", "lamb").max


//排序
val b = ArrayBuffer(1, 7, 2, 9)
//val bSorted = b.sorted(_ < _)//这种排序貌似是有问题的
val bSorted = b.sorted();


val a = Array(1, 7, 2, 9)
//quickSort返回值是空，是对a数组进行排序
scala.util.Sorting.quickSort(a)


a.mkString(" and ")
// "1 and 2 and 7 and 9"
a.mkString("<", ",", ">")
// "<1,2,7,9>"


```

### 多维数组

```scala
//创建数组
val matrix = Array.ofDim[Int](2,4);

//赋值某个元素
matrix(0) = Array(0,2,3,4);

//取值
matrix(0)(2)

//创建数组
var matrix01 = new Array[Array[Int]](4);

//赋值某个元素
matrix01[0] = Array(0,2,3,4,5,7);

//如果取未赋值的时候，会报错java.lang.NullPointerException

//遍历
for (arr <- matrix01) {println(arr)};
//或者
for (i <- 0 until matrix01.length) {println(matrix01(i))};


```

## 3.8 与java的互操作
略