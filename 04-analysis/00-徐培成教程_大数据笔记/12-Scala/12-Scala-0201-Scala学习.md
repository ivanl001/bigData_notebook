主要看代码

> 昨日学习总结

```scala
object  //静态成员，里面的属性相当于全局变量， 方法相当于静态方法， mian方法需要定义在object中

class   //就是类

trait   //接口

//方法或者函数定义一
def function(paramA: Int):Unit = {

}
//方法或者函数定义二
def funciton(paramA: Int):Unit{

}

var    //变量
val    //常量


map("a" -> "b", "key" -> "value")

("string", 2, 2.3)    //tuple, 最多22个元素好像是

//for循环
for(i <- 0 to 10){

}


Unit  ====等价于java中的void

javac //编译.java文件
javap //查看java的.class文件

scalac  //编译scala文件
javap   //scala文件编译后也是.class文件， 用javap即可查看


val arr01  = new Array(100);//这里是创建有100个元素的数组
val arr02 = Array(100); //这个是创建只有100这个Int值的数组，其实调用的是apply方法


"hello"(1)    //也是调用apply方法,获取第二个字母

//伴生对象
companion object  //和类同名，必须定义在一个scala文件中


//主构造器，在类名后买呢直接添加被构造的参数，@BeanProperty可以自动的生成getter和setter方法，类中方法默认好像是私有的，类中的println(name)是可以直接别执行的
import scala.beans.BeanProperty

class Person03 (@BeanProperty var name:String, @BeanProperty var age:Int, @BeanProperty var num:Int){

  println(name)

  def printf(): Unit ={
    println("name:" + name + "age:" + age +"num:" + num)
  }

  /*def  this(name:String, age:Int ,num:Int){
    this(name, age, num)
    this.name = name
    this.age = age+2
    this.num = num + 100
  }*/

  def getTheInfo(): Unit ={
    println("name:" + name + "age:" + age +"num:" + num)
    println("name:" + this.name + "age:" + this.age +"num:" + this.num+100)
  }
}

//辅助构造器，构造器中需要调用this()这个父类方法,参数是需要初始化的，同swift类似
import scala.beans.BeanProperty

class Person02 {

  @BeanProperty var name = ""
  @BeanProperty var num=0
  @BeanProperty var age = 30

  println("ivanl001" + this.age)

  //下面这种都是辅助构造器
  def this(name: String){
    this()
    this.name = name
  }

  def this(age: Int){
    this()
    this.age = age
  }

  def this(name:String, age: Int){
    this()
    this.name = name
    this.age = age
  }

}

```