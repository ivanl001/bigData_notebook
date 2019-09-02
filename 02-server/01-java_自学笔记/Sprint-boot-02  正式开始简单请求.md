## 1，版本选择
* 正如01中所说的，使用的时候如果有一些莫名其妙的问题，怎么都解决不掉的，可以尝试以下更换另外一个版本试一下，有时候可能真的是版本的问题或者其他我们自己不太好解决的问题。换个版本就好了。目前使用1.5.18版本ok，其他版本使用正常的下载方式导入到idea中好像都有问题，不清楚具体是我环境的问题还是版本内在的问题

## 2，使用步骤
### 0201, 下载方式一：官网下载
* 下载时候如果有需要的依赖可以搜索选中，也可以打开最下面“Switch to the full version”选择所需要的依赖。当然也可以直接下载，之后在pom.xml中添加依赖
  
  > https://start.spring.io/

### 0202, 下载方式二：idea下载
*同样，在后面的页面可以选择版本*

## 3，使用
### 0301，SprintbootApplication
* SprintbootApplications相当于main函数，通过这里运行，idea中可以直接启动spring项目

* 做一个简单的请求，注意：类名前必须要加@RestController注解，否则请求不通的
  ```java
  package im.ivanl001.CallLogWeb;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RestController;
  
  /**
   * #author      : ivanl001
   * #creator     : 2018-12-15 20:36
   * #description : 测试类
   **/
  @RestController
  public class TestPage {
      @RequestMapping("/ivanl001")
      public String ivanl001(){
          String str = "ivanl001 is the king of world!";
          System.out.println(str);
          return str;
      }
  }
  ```

## 4, 项目启动的方式
### 0401, idea启动
### 0402，maven启动
  * 在项目的根目录下：用如下命令进行启动：*
    
    > mvn spring-boot:run

## 5,打成可运行的jar包

* 00, 查看当前的项目maven依赖架包之间的关系

  *在pom.xml文件目录中执行如下命令* 
  
  > mvn dependency:tree


* 01, 使用的是1.5.18版本，按照文档直接打包的时候报错：搜索之后发现好像需要添加一个插件，具体如下：
  ```java
  [INFO] ------------------------------------------------------------------------
  [ERROR] Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.18.1:test (default-test) on project sprintboot: Execution default-test of goal org.apache.maven.plugins:maven-surefire-plugin:2.18.1:test failed: There was an error in the forked process
  [ERROR] org.apache.maven.surefire.util.SurefireReflectionException: java.lang.ClassNotFoundException: org.apache.maven.surefire.junit4.JUnit4Provider
  [ERROR] 	at org.apache.maven.surefire.util.ReflectionUtils.loadClass(ReflectionUtils.java:252)
  [ERROR] 	at org.apache.maven.surefire.util.ReflectionUtils.instantiateOneArg(ReflectionUtils.java:128)
  [ERROR] 	at org.apache.maven.surefire.booter.ForkedBooter.createProviderInCurrentClassloader(ForkedBooter.java:230)
  [ERROR] 	at org.apache.maven.surefire.booter.ForkedBooter.invokeProviderInSameClassLoader(ForkedBooter.java:199)
  [ERROR] 	at org.apache.maven.surefire.booter.ForkedBooter.runSuitesInProcess(ForkedBooter.java:155)
  [ERROR] 	at org.apache.maven.surefire.booter.ForkedBooter.main(ForkedBooter.java:103)
  [ERROR] Caused by: java.lang.ClassNotFoundException: org.apache.maven.surefire.junit4.JUnit4Provider
  [ERROR] 	at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
  [ERROR] 	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
  [ERROR] 	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:335)
  [ERROR] 	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
  [ERROR] 	at org.apache.maven.surefire.util.ReflectionUtils.loadClass(ReflectionUtils.java:244)
  [ERROR] 	... 5 more
  ```

* 02, 在pom.xml中添加：
  ```xml
  <plugin>
  	<groupId>org.apache.maven.plugins</groupId>
  	<artifactId>maven-surefire-plugin</artifactId>
  	<version>2.18.1</version>
  	<configuration>
  		<skipTests>true</skipTests>
  	</configuration>
  </plugin>
  ```
* 03, 然后重新打包可以
  
> mvn package
  
* 04, 查看打的jar包中的内容
  
> jar tvf target/calllog.jar
  
* 05，运行jar包
  
  > java -jar /target/calllog.jar