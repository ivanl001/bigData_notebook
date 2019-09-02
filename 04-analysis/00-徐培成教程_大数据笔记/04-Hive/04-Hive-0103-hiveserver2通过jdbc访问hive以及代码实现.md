*前面的基本使用都是在hive的命令端使用的，如果想要在其他非本机平台或者代码中访问hive数据库，那么需要启动hiveserver2通过监听10000端口实现jdbc访问数据库,具体如下:*



## 1，hiveserver2

  * 01，启动hiveserver2
    *也可以到hive到bin目录下直接运行hiveserver2命令*
    

```shell
hive --service hiveserver2 &
hive/bin/hiveserver2
```



  * 02，查看是否成功，如果成功会监听10000端口
    

```shell
netstat -anop | grep 10000
netstat -lntp | grep 10000
```



## 2, hive的Java API接口

* 01, 简单连接并查询


```java
package im.ivanl001.bigData.Hive;

import java.sql.*;

/**
 * #author      : ivanl001
 * #creator     : 2018-11-05 12:18
 * #description : hive的基本使用
 **/
public class A01_Hive_connect_and_use {

    private static String driverName = "org.apache.hive.jdbc.HiveDriver";

    public static void main(String[] args) throws Exception {

        //1，加载类
        Class.forName(driverName);

        //2, 获取连接
        //有一点需要注意的是：10000端口是beeserver2的端口，所以必须要在hive服务端开启beeserver2，要不然是没法连接上的哈
        //整理的账号密码请随意填写，没有关系
        Connection connection = DriverManager.getConnection("jdbc:hive2://centos01:10000/gmall", null, null);

        //3, 准备语意，进行查询
        Statement statement = connection.createStatement();
        ResultSet resultSet = statement.executeQuery("select * from gmall.ads_uv_count");
        while (resultSet.next()) {

            System.out.println("ivanl001");

            System.out.println(resultSet.getString("dt"));
            System.out.println(resultSet.getString("day_count"));
            System.out.println(resultSet.getString("wk_count"));
            System.out.println(resultSet.getString("mn_count"));
            System.out.println(resultSet.getString("is_weekend"));
            System.out.println(resultSet.getString("is_monthend"));
        }

        //4, 关闭
        statement.close();
        connection.close();
    }
}
```

