## 1, 使用PreparedStatement提高性能

* PreparedStatement比Statement要好一点
* 因为可以放置sql注入

## 2, 使用executeBatch进一步提高性能

* 使用PreparedStatement的时候：
* 如果情况允许的下：
* 关闭自动提交connection.setAutoCommit(false);
* 使用批处理：
* addBatch()和executeBatch

⚠️：connection.commit()是对mysql的设置，只能控制事务， 也就是说，如果每1000条数据提交一次，那么这1000条数据会在1个事务中，不会频繁的创建关闭事务。

但是这1000条数据还是会一条一条的发送给mysql服务器。

⚠️：addBatch()和executeBatch则是对应用程序的设置，也就是说，如果每1000条数据addBatch一次，然后executeBatch，那么会1000条一起发送给服务器。如果使用这个的话，里面已经提交了，所以也就没必须要在调用提交的方法了，如下： 

```java
//插入, 这里是批量，可以放置sql注入， 关闭自动提交是为了不需要每次插入都要创建一个事务。
//这里用preparedStatement.addBatch();的作用是做一个本地缓存，满足了再统一发给mysql。如果自动提交，就会直接事务处理，如果是手动提交，那么先不处理，等提交等时候一起加在一个事务里
@Test
public void preparedStatementBatch() throws Exception{
  String driverClass = "com.mysql.jdbc.Driver";
  String url = "jdbc:mysql://centos01:3306/test";
  String username = "root";
  String password = ",.";

  Class.forName(driverClass);
  Connection connection = DriverManager.getConnection(url, username, password);
  connection.setAutoCommit(false);//关闭自动提交

  String sqlStr = "insert into users (email, psd) values (?,?)";

  PreparedStatement preparedStatement = connection.prepareStatement(sqlStr);

  preparedStatement.setString(1, "13918667287@139.com");
  preparedStatement.setString(2, "0000");

  Long current_start = System.currentTimeMillis();
  for (int i = 0; i < 100000; i++) {
    preparedStatement.addBatch();
    if (i % 1234 == 0) {
      preparedStatement.executeBatch();
    }
  }

  //里面不一定能整除，所以出来还是要做一下批处理执行
  preparedStatement.executeBatch();
  connection.commit();

  Long current_end = System.currentTimeMillis();
  System.out.println("costTime: " + (current_end-current_start));
  //costTime: 3342, 好那么一丢丢
  //如果从一万增加到十万， 18286，

  preparedStatement.close();
  connection.close();
}
```



## 3, 使用存储过程更好提高性能

* 存储过程和函数的区别在于函数有返回值，存储过程则是没有返回值的
* 具体看03-Hadoop-1002-mysql-存储过程和函数内容