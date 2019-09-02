### 第五章: Object Oriented Programming

### 第五章 类（比较抽象，略过）
### 第六章 对象（比较抽象，略过）
### 第七章 包和引入（比较抽象，略过）
### 第八章 继承（比较抽象，略过）

*看书和代码吧，没啥好写的，索性把教程中第一天的所有笔记拷贝一份记录一下*

```scala
scala
-------------
	java语言的脚本化。
 
REPL
-----------------
	read + evaluate + print + loop

安装scala解释程序
------------------
	1.scala-2.12.1.msi
	2.进入scala命令行

		//变量
		scala>var a = 100			//变量

		//常量
		scala>val a = 100			//常量，不能重新赋值。

		//定义类型
		scala>val a:String = "hello" ;
		scala>a = "world"			//wrong


	//操作符重载 _ $
	scala>1 + 2
	scala>1.+(2)					//


	//scala函数,没有对象.
	//scala方法，通过对象调用。
	scala>import scala.math._		//_ ===> *
	scala>min(1,2)

	//
	scala>1.toString				//方法
	scala>1.toString()				//方法形式
	scala>1 toString				//运算符方式

	//apply
	scala>"hello".apply(1)			//等价于xxx.apply()
	scala>"hello"(1)				//

	//条件表达式,scala的表达式有值,是最后一条语句的值。
	scala>val x = 1 ;
	scala>val b = if x > 0 1 else -1 ;

	//Any 是Int和String的超类。

	//类型转换
	scala>1.toString()
	scala>"100".toInt()				//

	//空值
	scala>val y = (x = 1)			//y:Unit= ()类似于java void.
		
	
	//粘贴复制
	scala>:paste
			....
		ctrl + d					//结束粘贴模式

        javac               java
		*.java --------> *.class  -------->程序 


	//输出
	scala>print("hello")
	scala>println("hello")
	scala>printf("name is %s , age is %d", "tom",12);
	//读行
	scala>val password = readLine("请输入密码 : ") ;

	//查看帮助
	scala>:help
	

	
	//循环
	scala>:paste
		var i = 0 ;
		while(i < 10 ){
			println(i) ;
			i += 1;
		}

	//99表格
	scala>:paste
		var row = 1 ; 
		while(row <= 9 ){
			var col = 1 ; 
			while(col <= row){
				printf("%d x %d = %d\t",col,row,(row * col)) ;
				col += 1 ;
			}
			println();
			row += 1 ;
		}
	
	//百钱买白鸡问题.
		100块钱100只鸡。
		公鸡:5块/只
		母鸡:3块/只
		小鸡:1块/3只
	
	//公鸡
	var cock = 0 ;
	while(cock <= 20){
		//母鸡
		var hen = 0 ;
		while(hen <= 100/3){
			var chicken = 0 ;
			while(chicken <= 100){
				var money = cock * 5 + hen * 3 + chicken / 3 ;
				var mount = cock + hen + chicken ;
				if(money == 100 && mount == 100){
					println("cock : %d , hen : %d , chicken : %d",cock,hen,chicken) ;
				}
			}
		}
	}

	//for循环
	//to []
	scala>for (x <- 1 to 10){
		println(x) ;
	}

	//until [1,...10)
	scala>for (x <- 1 10){
		println(x) ;
	}

	//scala没有break continue语句。可以使用Breaks对象的break()方法。
	scala>import scala.util.control.Breaks._
	scala>for(x <- 1 to ) {break() ; print(x)} ;


	//for循环高级
	//双循环,守卫条件
	scala>for(i <- 1 to 3 ; j <- 1 to 4 if i != j ) {printf("i = %d, j = %d , res = %d ",i,j,i*j);println()} ;	
	
	//yield，是循环中处理每个元素，产生新集合
	scala>for (x <- 1 to 10 ) yield x % 2 ; 


	//定义函数
	def add(a:Int,b:Int):Int = {
	var c = a + b  ;
	return c  ;
	}

	scala>def add(a:Int,b:Int):Int =	a + b 

	//scala实现递归 n! = n * (n - 1)!
	4!  = 4 x 3!
	4!  = 4 x 3 x 2!
	4!  = 4 x 3 x 2 x 1!
	
	//递归函数必须显式定义返回类型
	scala>def fac(n:Int):Int = if(n ==1 ) 1 else n * fac(n-1) ;


	//函数的默认值和命名参数
	scala>def decorate(prefix:String = "[[",str:String,suffix:String = "]]") = {
		prefix + str + suffix 
	}

	scala>decorate(str="hello")
	scala>decorate(str="hello",prefix="<<")

	//变长参数
	scala>def sum(a:Int*) = {
		var s = 0 ;
		for (x <- a) s += x;
		s
		}

	scala>add(1 to 10)			//wrong
	scala>add(1 to 10:_*)		//将1 to 10当做序列处理。
	scala>def sum(args:Int*):Int = {if (args.length == 0) 0 else args.head + sum(args.tail:_*)}

	//过程,没有返回值，没有=号。
	scala>def out(a:Int){
		println(a) ;
	}

	//lazy延迟计算
	scala>lazy val x = scala.io.Source.fromFile("d:/scala/buy.scala00").mkString()
			x:<lazy>
			x

	//异常
	scala>
		try{
		  "hello".toInt;
		}
		catch{					//交给
			case _:Exception    => print("xxxx") ;
			case ex:java.io.IOException => print(ex)
		}


_的意义
---------------------
	1.统配相当于*
	2.1 to 10 :_*	,转成序列
	3.case _:Exception    => print("xxxx") ;

数组(定长)
---------------
	java : int[] arr = int int[4] ;
	scala>var arr = new Array[Int](10);			//apply(10)
	scala>var arr = Array(1,2,3,4,);			//推断
	scala>arr(0)								//按照下标访问元素

变长数组
-------------------
	scala>import scala.collection.mutable.ArrayBuffer
	scala>val buf = ArrayBuffer[Int]();			//创建数组缓冲区对象

	//+=在末尾追加
	scala>buf += 1

	//操纵集合
	scala>buf ++= ...

	//trimEnd,从末尾移除元素
	scala>buf.trimStart(2)
	scala>buf.trimEnd(2)

	//insert,在0元素位置插入后续数据
	scala>buf.insert(0,1,2)
	
	//remove按照索引移除
	scala>buf.remove(0)

	//toArray
	scala>buf.toArray

	//数组操作
	scala>for (x <- 1 to 10 if x % 2 ==0) yield x * 2	//
	scala>var a = Array(1 to 10)

	//数组常用方法
	scala>arr.sum
	scala>arr.min
	scala>arr.max

	
	//排序
	scala>import scala.util.Sorting._
	scala>val arr = Array(1,4,3,2)
	scala>quickSort(arr)				//arr有序

	//Array.mkString
	scala>arr.mkString("<<",",",">>")	//<<1,2,3,4>>

	//多维数组
	scala>var arr:Array[Int] =new Array[Int](4);
	//二维数组,3行4列
	scala>val arr = Array.ofDim[Int](3,4)
	//下标访问数组元素
	scala>arr(0)(1)
	scala>arr.length


	//和java对象交互，导入转换类型,使用的隐式转换
	scala>import scala.collection.JavaConversions.bufferAsJavaList
	scala>val buf = ArrayBuffer(1,2,3,4);
	scala>val list:java.util.List[Int] = buf ;

 
	//映射和元组
	//key->value
	//scala.collection.immutable.Map[Int,String] =不可变集合
	scala>val map = Map(100->"tom",200->"tomas",300->"tomasLee")
	//通过key访问value
	scala>map(100)
	scala>val newmap = map + (4->"ttt")
	
	//可变
	scala>val map = new scala.collection.mutable.HashMap[Int,Int]
	scala>val map = scala.collection.mutable.HashMap[Int,Int]()
	scala>map += (1->100,2->200)	//追加
	scala>map -= 8					//移除元素

	//迭代map
	scala>for ((k,v)<- map) println(k + ":::" + v);

	//使用yield操作进行倒排序(kv对调),
	scala>for ((k,v)<- map) yield (v,k);

	//元组tuple,元数最多22-->Tuple22
	scala>val t = (1,"tom",12) ;

	//访问元组指定元
	scala>t._2
	scala>t _2
	//直接取出元组中的各分量
	scala>val (a,b,c) = t		//a=1,b="tom",c=12
	
	//数组的zip,
	//西门庆 -> 潘金莲  牛郎 -> 侄女  ,
	scala>val hus = Array(1,2,3);
	scala>val wife = Array(4,5,6);
	scala>hus.zip(wife)		//(1,4),(2,5),(3,6)


OOP
----------------
	scala>class Person{
		//定义变量,私有类型,必须初始化
		//set/get也私有
		private var id = 0 ;
		
		//只有get方法，没有set方法
		val age = 100 ;

		//生成私有属性，和共有的get/set方法。
		var name = "tom" ;
		//默认public
		def incre(a:Int) = {id += a ;}
		//如果定义时，没有(),调用就不能加()
		def current() = id 
	}

	scala>var p = new Person();
	scala>p.current()
	scala>p.current
	scala>p.incr(100)
	scala>p.name
	scala>p.name_=("kkkk")
	scala>p.name = "kkkk"


private[this]作用,控制成员只能在自己的对象中访问。
-------------------
	class Counter{
		private[this] var value =  0 ;
		def incre(n:Int){value += n}

		def isLess(other:Counter) = value < other.value ;
	}

定义BeanProperty注解
----------------------
	class Person{
		@scala.reflect.BeanProperty
		var  name:String  = _
	}


构造函数
----------------
	主构造器		//
	辅助构造		//
	
	class Person{
		var id = 1 ;
		var name = "tom" ;
		var age = 12;

		//辅助构造
		def this(name:String){
			this();
			this.name = name ;
		}
		//辅助构造
		def this(name:String,age:Int){
			//调用辅助构造
			this(name) ;
			this.age = age ;
		}
	}

	
	//主构造
	//val ===> 只读
	//var ==> get/set
	//none ==> none
	class Person(val name:String,var age:Int , id :Int){
		
		def hello() = println(id)
	}


Object
-----------------
	说明：scala没有静态的概念，如果需要定义静态成员，可以通过object实现。
	      编译完成后，会生成对应的类，方法都是静态方法，非静态成员对应到单例类中
		  单例类以Util$作为类名称。

	scala>object Util{
		//单例类中.(Util$)
		private var brand = "benz" ;
		//静态方法.
		def hello() = println("hello world");
	}

伴生对象(companions object)
--------------------------
	类名和object名称相同，而且必须在一个scala文件中定义。
	class Car{
		def stop() = println("stop....")
	}

	object Car{
		def run() = println("run...")
	}

抽象类
--------------
	//定义抽象类
	abstract class Dog{
		def a():Unit
	}


object等价于java中的静态。
---------------------------
	object Jing8 extends Dog{
		//重写方法
		override def a():Unit= print("hello") ;
	}

object Util{
	def apply(s:String) = println(s) ;
}

Util("hello world");
Util.apply("hello world");

trait
----------
	特质，等价于java中的接口.


scalac编译scala文件，产生class文件。
------------------------------------
	cmd>scalac xxxx.scala

运行class程序
--------------------
	cmd>scala Person

一步到位
-------------------
	cmd>scala Person.scala

idea中安装scala插件
------------------------



trait
-----------
	traint HelloService{
	}


//包对象，编译完之后生成以xxx为package，下面含有类package.class + package.class
package object xxxx{

}


//约束可见性。
-----------------------
private[package|this]

导入
--------------
import java.io.Exception
import java.io.{A,B,C}			//
import java.io.{A => A0}		//别名


```

