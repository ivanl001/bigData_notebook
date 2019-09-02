## 1, 首先在application.properties中配置两个属性

```properties
# 设置日志
logging.file=estore_web
# 注意：这个path配置是lagback的配置，sprigboot默认不识别，需要配置logback-spring.xml才行
# 为什么不配置logback.xml,因为logback.xml会先application.properties加载，而logback-spring.xml会后于application.properties加载，这样我们在application.properties文中设置日志文件名称和文件路径才能生效。
logging.path=/Users/ivanl001/Desktop/bigData/springboot/logs/
```

## 2, 在资源根目录下生成logback-spring.xml文件，配置如下

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>

    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <!--我已经在application.properties中定义过了，所以这里不用-->
    <!--<property name="LOG_HOME" value="/data/springboot/logs" />-->



    <!--这里相当于定义一个STDOUT的控制台输出模版，后面会引用-->
    <!-- 控制台输出 -->
    <!-- %m输出的信息,%p日志级别,%t线程名,%d日期,%c类的全名,%i索引【从数字0开始递增】,,, -->
    <!-- appender是configuration的子节点，是负责写日志的组件。 -->
    <!-- ConsoleAppender：把日志输出到控制台 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>


    <!--这里相当于定义一个文件滚动模版，后面会引用-->
    <!-- 按照每天生成日志文件 -->
    <appender name="FILE"  class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <!--<FileNamePattern>${LOG_PATH}/${LOG_FILE}.%d{yyyy-MM-dd}.log</FileNamePattern>-->
            <!--这个是上面的完善版，按月归档，按天滚动-->
            <fileNamePattern>${LOG_PATH}/%d{yyyy-MM,aux}/${LOG_FILE}.%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>

            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- 除按日志记录之外，还配置了日志文件不能超过10M(默认)，若超过10M，日志文件会以索引0开始， -->
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>

        </rollingPolicy>

        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>

        <!--日志文件最大的大小,写这个好像不对-->
        <!--<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>1MB</MaxFileSize>
        </triggeringPolicy>-->

    </appender>



    <!--myibatis log configure-->
    <!--<logger name="com.apache.ibatis" level="TRACE"/>
    <logger name="java.sql.Connection" level="DEBUG"/>
    <logger name="java.sql.Statement" level="DEBUG"/>
    <logger name="java.sql.PreparedStatement" level="DEBUG"/>-->


    <!-- 日志输出级别 -->
    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
    </root>


    <!--日志异步到数据库 -->
    <!--<appender name="DB" class="ch.qos.logback.classic.db.DBAppender">
        &lt;!&ndash;日志异步到数据库 &ndash;&gt;
        <connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">
            &lt;!&ndash;连接池 &ndash;&gt;
            <dataSource class="com.mchange.v2.c3p0.ComboPooledDataSource">
                <driverClass>com.mysql.jdbc.Driver</driverClass>
                <url>jdbc:mysql://127.0.0.1:3306/databaseName</url>
                <user>root</user>
                <password>root</password>
            </dataSource>
        </connectionSource>
    </appender>-->


</configuration>

```

## 重启项目即可