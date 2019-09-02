*使用idea的时候，会自动编译，idea用maven打包是把编译的class文件放进包中，所以scala代码在idea的时候大多可以打包正常。但是有时候如果编译不及时的时候，就会导致打包有点问题，包里内容不是最新的。解决办法：添加如下maven插件：*
```xml
<plugins>
    <!--这里指定编译器，默认是1.5，这里可以更改为1.8-->
    <!--Maven default language level is 1.5 (5.0), you will see this version as the Module language level on the screenshot above.-->
    <!--    This can be changed using maven-compiler-plugin configuration inside pom.xml: -->
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>

    <!--这里指定编译scala的插件，就算是关闭了idea的自动编译，scala也可以编译-->
    <plugin>
        <groupId>net.alchim31.maven</groupId>
        <artifactId>scala-maven-plugin</artifactId>
        <version>3.2.2</version>
        <configuration>
            <recompileMode>incremental</recompileMode>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <goal>compile</goal>
                    <goal>testCompile</goal>
                </goals>
            </execution>
        </executions>
    </plugin>

</plugins>
```