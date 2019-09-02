* 编译scala源码
  
> scalac Animal.scala
  
* 查看编译后的class文件
  > javap Cat.class
  > javap -private Cat.class



* 01, 解释一下：在大数据中，或者scala中，数据是主题，所以总是在处理数据的 时候把函数传递进入，比如说在map中传递一个匿名函数，然后处理map中的每一个数据，比如Array(1,2,3).map{(x: Int) => x*x}， 这和java中把数据传递给方法是不同的哈
* 
* 主要看代码

