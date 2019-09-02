# JDBC的基本操作

* 更多代码参考：im.ivanl001.bigData.a00_database

## 1, JDBC的CRUD操作

### 1.1, 增

```java
@Test
public void jdbc_write() throws Exception {
	
  //0, 加载类
  Class.forName("com.mysql.jdbc.Driver");
  
  //1, 获取连接
  Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "ivanl48265");

  //2,准备语意
  PreparedStatement preparedStatement = connection.prepareStatement("INSERT  into tb_user (username, password, phone, created, updated) values (?, ?, ?, ?, ?)");
  preparedStatement.setString(1, "ivanl003");
  preparedStatement.setString(2, "ivanl002");
  preparedStatement.setString(3, "17612156404");

  //3,设置参数
  preparedStatement.setDate(4, new Date(System.currentTimeMillis()));
  preparedStatement.setDate(5, new Date(System.currentTimeMillis()));
	
  //4,执行语句
  preparedStatement.executeUpdate();

  //5, 关闭语意和连接
  preparedStatement.close();
  connection.close();
}
```

### 1.2, 查

```java
@Test
public void jdbc_read() throws Exception {
  
  //0, 加载类
  Class.forName("com.mysql.jdbc.Driver");
  
  //1, 获取连接
  Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "ivanl48265");

  //2,准备语意
  PreparedStatement preparedStatement = connection.prepareStatement("select  * from tb_user");
  
  //4,执行语句
  ResultSet resultSet = preparedStatement.executeQuery();
  while (resultSet.next()) {
    int id = resultSet.getInt("id");
    String username = resultSet.getString("username");
    String password = resultSet.getString("password");
    String phone = resultSet.getString(4);
    String email = resultSet.getString(5);
    //不论是getTimestamp还是getString最后总是会多一个.0, 而我们需要的精度到秒就可以了
    //Timestamp date01 = resultSet.getTimestamp(6);
    Date date01 = resultSet.getDate(6);
    Time time01 = resultSet.getTime(6);
    String date02 = resultSet.getString(7);

    System.out.println(id + ", " + username + ", " + password + ", " + phone + ", " + email + ", " + date01 + " " + time01 + ", " + date02);
    //7, ivanl002, ivanl002, 17612156406, null, 2018-10-27 00:00:00, 2018-10-27 00:00:00.0
  }

  //5, 关闭语意和连接
  preparedStatement.close();
  connection.close();
}
```

### 1.3, 改

* 改动和上面的增查是一样的，更换一下语句即可

```java

```

### 1.4, 删

* 删除和上面的增查是一样的，更换一下语句即可

```java

```