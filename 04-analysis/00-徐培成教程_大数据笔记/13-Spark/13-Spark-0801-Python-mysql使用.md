## 1, python3.6使用mysql安装需要插件
*对于python3的话，如果想要连接mysql，需要安装PyMySQL插件，通过pip3安装后， 会自动的安装到python3的插件文件夹中可以在代码中直接进行调用的*
* pip3 install PyMySQL

## 2, 连接测试 select version()
```python
#
# author      : ivanl001
# creator     : 2018-12-02 19:30
# description : select version()
#

import pymysql

# 1, 打开数据库
db = pymysql.connect("localhost","root","ivanl48265","test")

# 2, 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 3, 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT version()")

# 4, 使用 fetchone() 方法获取单条数据.
data = cursor.fetchone()

# 5, 关闭数据库连接
db.close()

# 6, 解析数据
print ("Database version : %s " % data)

```

## 3, 创表 create table
```python
#
# author      : ivanl001
# creator     : 2018-12-06 14:42
# description : create table
#
import pymysql

# 打开数据库连接
db = pymysql.connect("localhost","root","ivanl48265","test")

# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 使用 execute() 方法执行 SQL，如果表存在则删除
cursor.execute("drop table if EXISTS employee")

# 使用预处理语句创建表
sql = """CREATE TABLE employee (
         FIRST_NAME  CHAR(20) NOT NULL,
         LAST_NAME  CHAR(20),
         AGE INT,  
         SEX CHAR(1),
         INCOME FLOAT )"""

cursor.execute(sql)

# 关闭数据库连接
db.close()
```

## 4, 插入 insert into
```python
#
# author      : ivanl001
# creator     : 2018-12-06 14:46
# description : insert into
# 

import pymysql

# 打开数据库连接
db = pymysql.connect("localhost","root","ivanl48265","test")

# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# SQL 插入语句
sql = """INSERT INTO EMPLOYEE(FIRST_NAME,
         LAST_NAME, AGE, SEX, INCOME)
         VALUES ('Mac', 'Mohan', 20, 'M', 2000)"""

# sql = "INSERT INTO EMPLOYEE(FIRST_NAME, \
#        LAST_NAME, AGE, SEX, INCOME) \
#        VALUES (%s, %s,  %s,  %s,  %s )" % \
#       ('Mac', 'Mohan', 20, 'M', 2000)

try:
    # 执行sql语句
    cursor.execute(sql)
    # 提交到数据库执行
    db.commit()
except:
    # 如果发生错误则回滚
    db.rollback()

# 关闭数据库连接
db.close()
```

## 5，单条读取操作fetchone

```python
#
# author      : ivanl001
# creator     : 2018-12-02 19:30
# description : 
#

import pymysql

# 1, 打开数据库
db = pymysql.connect("localhost","root","ivanl48265","test")

# 2, 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 3, 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT * from users")

# 4, 使用 fetchone() 方法获取单条数据.
data = cursor.fetchone()

# 5, 关闭数据库连接
db.close()

# 6, 解析数据
id = data[0]
email = data[1]
password = data[2]

# print ("Database version : %s " % data)
print(str(id) + "-------:" + email + "------:" + password)

```

## 6, fetchall

```python
#
# author      : ivanl001
# creator     : 2018-12-06 14:36
# description : fetchall
#

import pymysql

# 1, 打开数据库
db = pymysql.connect("localhost","root","ivanl48265","test")

# 2, 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 3, 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT * from users")

# 4, 使用 fetchone() 方法获取单条数据.
data = cursor.fetchall()

# 5, 关闭数据库连接
db.close()

# 6, 解析数据
for row in data:

    id = row[0]
    email = row[1]
    password = row[2]

    # print ("Database version : %s " % data)
    print(str(id) + "-------:" + email + "------:" + password)

```

## 7, 更新update
```python
#
# author      : ivanl001
# creator     : 2018-12-06 14:53
# description : update
#

import pymysql

# 1, 打开数据库
db = pymysql.connect("localhost","root","ivanl48265","test")

# 2, 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 3, 使用 execute()  方法执行 SQL 查询
sql = "UPDATE EMPLOYEE SET AGE = AGE + 1 WHERE SEX = '%c'" % ('M')


try:
    # 执行SQL语句
    rowcount=cursor.execute(sql)
    # 提交到数据库执行
    db.commit()
except:
    # 发生错误时回滚
    db.rollback()


# 4, 关闭数据库连接
db.close()

# 5, 查看影响行
print(rowcount)
```

## 8, 删除delete
```python
#
# author      : ivanl001
# creator     : 2018-12-06 15:01
# description : delete
#


import pymysql

# 1, 打开数据库
db = pymysql.connect("localhost","root","ivanl48265","test")

# 2, 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 3, 使用 execute()  方法执行 SQL 查询
sql = "DELETE FROM EMPLOYEE WHERE AGE > %s" % (20)

try:
    # 执行SQL语句
    rowcount=cursor.execute(sql)
    # 提交到数据库执行
    db.commit()
except:
    # 发生错误时回滚
    db.rollback()

# 4, 关闭数据库连接
db.close()

# 5, 查看影响行
print(rowcount)
```

## 9, 手动提交事务

```python
#
# author      : ivanl001
# creator     : 2018-12-06 15:12
# description : 手动提交事物
#
import pymysql

# 1, 打开数据库
db = pymysql.connect("localhost","root","ivanl48265","test")
# 关闭事物的自动提交
db.autocommit(False)

try:
    # ------- 开启事物 -------
    db.begin()

    # 2, 使用 cursor() 方法创建一个游标对象 cursor
    cursor = db.cursor()

    # 3, 使用 execute()  方法执行 SQL 查询
    sql = "DELETE FROM EMPLOYEE WHERE AGE > %s" % (40)

    # 4, 执行SQL语句
    rowcount=cursor.execute(sql)

    # 5, 提交到数据库执行
    db.commit()

    # 6, 关闭游标
    cursor.close()

except:
    # 发生错误时回滚
    db.rollback()

# 4, 关闭数据库连接
db.close()

# 5, 查看影响行
print(rowcount)
```

## 10, count

```python
#
# author      : ivanl001
# creator     : 2018-12-06 15:21
# description : 
#

import pymysql

# 1, 打开数据库
db = pymysql.connect("localhost","root","ivanl48265","test")

# 2, 获取游标
cursor = db.cursor()

# 3, 执行
cursor.execute("select count(*) from employee")

# 4，关闭连接
db.close()

# 5, 获取结果并打印
count = cursor.fetchone()
print(count)
print(count[0])
```

## 11, 存储过程 procedure和函数的调用

```python
#
# author      : ivanl001
# creator     : 2018-12-06 15:26
# description :
#


import pymysql

# --------------------------调用存储过程------------------------------
# 1, 打开数据库
db = pymysql.connect("localhost","root","ivanl48265","test")

# 2, 获取游标
cursor = db.cursor()

# 3, 执行,这里是调用存储过程
cursor.execute("call batchInsert(1000)")

# 4，关闭连接
db.close()

# 5, 获取结果并打印
print("proceduce called")


# --------------------------调用函数------------------------------
# 1, 打开数据库
db = pymysql.connect("localhost","root","ivanl48265","test")

# 2, 获取游标
cursor = db.cursor()

# 3, 执行,这里是调用 函数
cursor.execute("select test.imSumFunc(10, 20)")

# 4，关闭连接
db.close()

# 5, 获取结果并打印
result = cursor.fetchone()
print(result)
print(result[0])
```