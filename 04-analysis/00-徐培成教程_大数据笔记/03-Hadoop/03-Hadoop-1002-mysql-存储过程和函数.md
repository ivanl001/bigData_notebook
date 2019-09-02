# 存储过程

> 存储过程和函数的区别是：存储过程没有返回值(但是可以给输出参数赋值)， 函数有返回值



## 0, 存储过程的简单使用

```shell
# 显示所有的存储过程及其状态
show procedure status;

# 显示test数据库中userCount这个存储过程的创建语句
show create procedure test.userCount;

# 调用存储过程
call test.userCount(@theCount)
# 选择结果值
select @theCount;


```



## 1, mysql中定义存储过程



### 1.1, 第一个简单的存储过程

```mysql
# 没有输入参数，只有一个返回值，返回users的数据条数
CREATE DEFINER=`root`@`%` PROCEDURE `userCount`(OUT param INT)
BEGIN
  #Routine body goes here...
	select count(1) from users;

END
```



```shell
# 调用存储过程
call test.userCount(@theCount)
# 选择结果值
select @theCount;
```





### 1.2, 有参数的加法存储过程

```shell
CREATE DEFINER=`root`@`%` PROCEDURE `imSum`(IN a INT, IN b INT, OUT z INT)
BEGIN
  #Routine body goes here...
	set z := a + b;
END
```



```shell
# 调用存储过程
CALL test.imSum(1,3, @theSum);

# 选择结果值
select @theSum;
```



## 2, java api调用存储过程

```java
/*
     * 我在数据库里定义了一个简单的存储过程，如下：
     *
     *CREATE DEFINER=`root`@`centos01` PROCEDURE `imSum`(in a INT, in b INT, out z INT)
        BEGIN
        set z := a + b;
        END
     * 传入两个值，返回两个值的和
     * 我们这里用java调用这个存储过程
     *
     */
@Test
public void procedureTest() throws Exception{

  String driverClass = "com.mysql.jdbc.Driver";
  String url = "jdbc:mysql://centos01:3306/test";
  String username = "root";
  String password = ",.";

  Class.forName(driverClass);
  Connection connection = DriverManager.getConnection(url, username, password);

  connection.setAutoCommit(false);

  CallableStatement callableStatement = connection.prepareCall("{call imSum(?, ?, ?)}");
  callableStatement.setInt(1, 10);
  callableStatement.setInt(2, 33);

  callableStatement.setInt(3, Types.INTEGER);

  callableStatement.execute();

  int resultSum = callableStatement.getInt(3);
  //connection.commit();//这里应该提交不提交都没有关系，因为没有真正的操作数据库的东西，不提交也无任何影响

  System.out.println(resultSum);

  callableStatement.close();
  connection.close();
}
```



# mysql函数



## 0, 函数的简单使用

```shell
# 显示所有的函数及其状态
show function status;

# 显示test数据库中userCount这个函数的创建语句
show create function test.imSumFunc;

# 调用函数
select test.imSumFunc(1,10)
```



## 1, mysql中定义函数

```mysql
CREATE DEFINER=`root`@`%` FUNCTION `imSumFunc`(a INT, b INT) RETURNS int(11)
BEGIN
  #Routine body goes here...
	return a + b;
END
```

调用函数

```shell
 # 显示所有的函数及其状态
show function status;

# 显示test数据库中userCount这个函数的创建语句
show create function test.imSumFunc;

# 调用函数
select test.imSumFunc(1,10)
```



### 2, java api调用函数

```java
//函数的调用
@Test
public void functionAddTest() throws Exception {

  String driverClass = "com.mysql.jdbc.Driver";
  String url = "jdbc:mysql://centos01:3306/test";
  String username = "root";
  String password = ",.";

  Class.forName(driverClass);
  Connection connection = DriverManager.getConnection(url, username, password);

  connection.setAutoCommit(false);


  //注意：这里的问好一定要是英文符号
  CallableStatement callableStatement = connection.prepareCall("{?= call imSumFunc(?,?)}");
  callableStatement.setInt(2, 3);
  callableStatement.setInt(3, 34);

  callableStatement.registerOutParameter(1, Types.INTEGER);

  callableStatement.execute();
  connection.commit();

  int sum = callableStatement.getInt(1);
  System.out.println(sum);

  callableStatement.close();
  connection.close();
}
```

