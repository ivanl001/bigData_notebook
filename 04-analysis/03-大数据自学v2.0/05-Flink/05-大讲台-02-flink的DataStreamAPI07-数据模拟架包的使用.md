### 1, 下载源码

https://github.com/CloudWise-OpenSource/Data-Processer

```shell
git clone https://github.com/CloudWise-OpenSource/Data-Processer.git
```



### 2, 用idea打开该项目, 然后本地安装

```shell
maven clean install
```



### 3, 在使用该模拟架包的项目中引入该架包

```xml
<!--模拟生产数据的一个工具包-->
<dependency>
  <groupId>com.cloudwise.toushibao</groupId>
  <artifactId>simulatedata-generator</artifactId>
  <version>0.0.1</version>
</dependency>
```



### 4, 在项目的根目录下创建dictionaries目录，目录下放我们编写的字典

```java
name=xiaoming|||hanmeimei|||lilei
reqUrl=http://www.abc.com/a/b/c|||http://www.def.com/d/e/f
tmp=$Func{intRand(1000000000, 1999999999)}
b=testVar
ot=$Func{intRand(2)}
```

其中一些术语：

```java
函数变量：模版和词典中以"$Func{"开头，以"}"结尾的字符串是一个函数变量。形如：$Func{intRand()}，其中，intRand()为内置函数。不支持函数嵌套

词典变量：模版中以"$Dic{"开头，以"}"结尾的字符串是一个词典变量。形如：$Dic{name},其中，name为词典文件中的一个词典名。

自定义变量：模版中以"$Var{"开头，以"}"结尾的字符串是一个自定义变量。形如：$Var{tmp}，其中，tmp是自定义变量名。自定义变量需要与函数变量或者词典变量联合使用，中间以"="隔开，且无空格。 定义方式：$Var{tmp}=$Func{doubleRand(0,10,2)}。引用方式是；$Var{tmp}
```



其中一些内置函数:

- long timestamp() 生成当前13位时间戳（ms）。
- int intRand() 生成int正随机整数。
- int intRand(Integer n) 生成0～n的随机整数。
- int intRand(Integer s, Integer e) 生成s～e的随机整数。
- long longRand() 生成long正随机整数。
- double doubleRand() 生成0～1.0的随机双精度浮点数。
- doubleRand(Integer s, Integer e, Integer n) 生成s～e的保留n位有效数字的浮点数。
- String uuid() 生成一个uuid。
- String numRand(Integer n) 生成n位随机数。
- String strRand(Integer n) 生成n位字符串，包括数字，大小写字母。





### 5, 写一个模版文件, test.tpl

* 如果不想定义一个模版文件，直接把字符串写死在代码中，传给第6步中的tplStr也可以的

```json
 {
    "infos":{
        "d_id": $Func{intRand()},
        "version": $Var{tmp}=$Func{doubleRand(0,10,2)},
        "os_type": "$Dic{ot}",
        "os": "$Var{tmp}",
        "test": "$Dic{b}"
        }
 }
```



### 6, 读取模版文件内容，然后

#### 6.1, 定义模板方式

```java
 //加载词典(只需执行一次即可)
 DicInitializer.init();
 //编辑模版
 String tplStr = " {
    "infos":{
        "d_id": $Func{intRand()},
        "version": $Var{tmp}=$Func{doubleRand(0,10,2)},
        "os_type": "$Dic{ot}",
        "os": "$Var{tmp}",
        "test": "$Dic{b}"
        }
 }";
 //创建模版分析器（一个模版new一个TemplateAnalyzer对象即可）
 TemplateAnalyzer testTplAnalyzer = new TemplateAnalyzer("这里随意，应该只是个名字表示，五十几用途", tplStr);
 //分析模版生成模拟数据
 String abc = testTplAnalyzer.analyse();
 //打印分析结果
 System.out.println(abc);
```

#### 6.2, 直接字符串写死方式

```java
 //加载词典(只需执行一次即可)
 DicInitializer.init();
 //编辑模版
 String tplStr = "这里是读取test.tpl的内容";
 //创建模版分析器（一个模版new一个TemplateAnalyzer对象即可）
 TemplateAnalyzer testTplAnalyzer = new TemplateAnalyzer("这里随意，应该只是个名字表示，五十几用途", tplStr);
 //分析模版生成模拟数据
 String abc = testTplAnalyzer.analyse();
 //打印分析结果
 System.out.println(abc);
```



### 7, 我本人写的一个类

```java
package im.ivanl001.streaming.a11_flink_project01.data_simulator;

import com.cloudwise.sdg.dic.DicInitializer;
import com.cloudwise.sdg.template.TemplateAnalyzer;

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

/**
 * #author      : ivanl001
 * #creator     : 2019-07-24 16:12
 * #description :
 **/
public class Simulator {

    static {
        //加载词典(只需执行一次即可)
        try {
            DicInitializer.init();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static String simulate(String templateName) {

        //读取模版内容
        //获取文件路径
        String pathStr = Class.class.getClass().getResource("/").getPath() + templateName;
        FileReader fileReader = null;
        try {
            fileReader = new FileReader(pathStr);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        StringBuilder builder = new StringBuilder();

        try {
            //读取文件信息
            char[] chars = new char[1024];
            int b = 0;
            while ((b = fileReader.read(chars)) != -1) {
                builder.append(new String(chars, 0, b));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        //System.out.println("结果是：" + builder.toString());

        //创建模版分析器（一个模版new一个TemplateAnalyzer对象即可）
        TemplateAnalyzer analyzer = new TemplateAnalyzer("", builder.toString());
        String simulateData = analyzer.analyse();
        return simulateData;

    }

  	//使用方式
    public static void main(String[] args) {
      	//test.tpl文件放在resources文件目录下即可
        String simulateData = Simulator.simulate("test.tpl");
        System.out.println(simulateData);
    }
}
```

