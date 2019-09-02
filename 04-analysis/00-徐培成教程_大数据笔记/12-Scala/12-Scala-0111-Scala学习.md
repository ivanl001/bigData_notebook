### 第四章 映射和元祖

*个人暂时简单理解，映射其实内部就是多个元祖*

## 4.1，构造映射

```scala
//构建映射
val stu = Map("name" -> "ivanl001", "age"->10, "score"->100);

//stu: scala.collection.immutable.Map[String,Any] = Map(name -> ivanl001, age -> 10, score -> 100)

//构造映射方法二
val score = Map(("name","ivanl001"), ("age",10), ("score",100));

//score: scala.collection.immutable.Map[String,Any] = Map(name -> ivanl001, age -> 10, score -> 100)

//构建可变映射
scala> val scores = new scala.collection.mutable.HashMap[String, Int];

scala> val scores = new scala.collection.mutable.HashMap[String, Int]();

scala> val scores = scala.collection.mutable.HashMap[String, Int]();

```

## 4.2，获取映射中的值

```scala
//01:
val info = stu.get("name")
var name = stu("name");

//02,在不确定取出的键值是否存在的时候
val info = stu.getOrElse("zhag", "ivanl0000")

```

## 4.3, 更新

`注意：更新操作只能适用于可变数组`

```scala
val scores = scala.collection.mutable.HashMap("zhang"->100, "li"->50);

scores("zhang") = 500;

scores += ("ivanl01"->100, "kaven01"->200);

//移除的时候只需要键值
scores -= ("ivanl");

//对于不可变数组而言， +=或者-=应该也可以使用， 但是会产生新的数组
```

## 4.4, 迭代映射

```scala
for((key, value) <- scores){println(key, value)};

for(value <- scores.values){println(value)};

for(key <- scores.keySet){println(key)};

//如果想要反转一个映射，类似把键值和值对换，可以通过yield
val revertScores = for((key, value) <- scores) yield (value, key);

```

## 4.5, 4.6略过

## 4.7，元祖

```scala
val t = (1, "zhang", 99);
t: (Int, String, Int) = (1,zhang,99)

//获取元祖的某个元素，注意：这了序号是从1开始，而不是0
t._2
t _2

// 下面这种方式可以直接取出元祖中的值并赋值给前面的变量
var (a, b, c) = t

/*
a: Int = 1
b: String = zhang
c: Int = 99
*/
```

## 4.8，拉链操作

`简单来讲其实是可以把数组转成多维数组的一种操作`

```scala
scala> val arr01 = Array("zhang", "buer", "ivanl001");

scala> val arr02 = Array("danfeng", "zhang", "ivanl002");

scala> val zipped = arr01.zip(arr02);

//zipped: Array[(String, String)] = Array((zhang,danfeng), (buer,zhang), (ivanl001,ivanl002))

```