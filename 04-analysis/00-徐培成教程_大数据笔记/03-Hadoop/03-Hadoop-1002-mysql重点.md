在jdbc中可以设置：connection.setAutoCommit(false);

这个有什么作用呢：



看如下代码

```java
String sqlStr = "insert into users (email, psd) values (?,?)";
PreparedStatement preparedStatement = connection.prepareStatement(sqlStr);

preparedStatement.setString(1, "13918667287@139.com");
preparedStatement.setString(2, "0000");

Long current_start = System.currentTimeMillis();
for (int i = 0; i < 100000; i++) {
  preparedStatement.execute();
}
```

比如说我们想要插入100000条数据， 那么如果按照默认的方式的话， 默认AutoCommit = true，也即是开启的。

那么每进入一次循环，也就是preparedStatement.execute();就会开启事务，插入，关闭事务，从而完成一次插入。



但是如果这样的话，效率是比较低的，那么有时候在符合需求的情况下：

如果想要比如说，每100条数据创建一个事物，然后提交100条数据，关闭事务这样子，也就相当于批量操作(可以类比IO流中的buffer，不是一个字节一个字节读取了，而是n个字节放到一个缓冲区中，buffer放满之后进行flush，写出即可)， 这样子效率会高：那么你需要：

其实这里和IO的buffer还是有区别的，buffer是本地缓冲，先不发送， 而这里还是会直接发送，只不过提交的时候先创建事务，真正的执行语句



1, 关闭自动提交

```java
//关闭自动提交
connection.setAutoCommit(false);
```



2, 在需要提交的地方提交，比如说循环100遍的时候

```java
//不用担心事务，commi前会自动开启事务，提交后会关闭事务
connection.commit();
```



3, 如果只想最后提交，比如说本案例的100000条提交一次即可，那么其实也不需要手动提交，因为close方法中会进行提交(也是类比IO流，关闭的时候会flush最后的剩余没有填满buffer的那些数据)

```java
preparedStatement.close();
```





上面的和IO还是有区别的，就是上面的还是会每一条都会发送给sql，只不过满足提交的时候，才会创建事务，真正的执行sql。

但是PreparedStatement里面有一个addBatch();方法，可以实现本地先缓存，提交batch的时候才会真正的发送到mysql上

具体如下：

```java
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



