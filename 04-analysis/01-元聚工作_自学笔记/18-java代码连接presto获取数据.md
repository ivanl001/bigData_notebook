https://blog.csdn.net/zzq900503/article/details/79460240





```xml
<dependency>
    <groupId>com.facebook.presto</groupId>
    <artifactId>presto-jdbc</artifactId>
    <version>0.196</version>
</dependency>
```

```java
package im.ivanl001.docker_test.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

/**
 * Author: ivanl001
 * Date: 2019/3/21 10:46
 * Desc:
 */
@RequestMapping("/presto")
@RestController
public class Ivanl002Presto {

    @RequestMapping("/test")
    public String prestoTest() throws Exception {

        Class.forName("com.facebook.presto.jdbc.PrestoDriver");
        String url = "jdbc:presto://10.200.3.10:48080/hive/aries";
        Connection connection =  DriverManager.getConnection(url, "hdfs", null);
        Statement stmt = connection.createStatement();
        ResultSet rs = stmt.executeQuery("show tables");
        while (rs.next()) {
            System.out.println(rs.getString(1));
        }
        String sql2 = "select * from result_mhd_and_qxs_consume limit 5";
        ResultSet rs2 = stmt.executeQuery(sql2);
        while(rs2.next()) {
            String str = rs2.getString(1);
            System.out.println(str);
        }
        rs.close();
        connection.close();

        return "ivanl001 is the king of world!";
    }
}
```

