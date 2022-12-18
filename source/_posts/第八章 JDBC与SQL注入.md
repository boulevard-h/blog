---
title:  数据库原理第八章 JDBC与SQL注入
date: 2022-12-17 22:24:23
categories: 数据库系统原理
tags: [数据库]
---

本章前面介绍了嵌入式SQL、过程化SQL、存储过程和函数、ODBC编程等，但是考试不做要求，暂且略过。

## JDBC编程

Java Database Connectivity，Java语言调用有关API来**连接数据库、发送SQL语句、获取查询结果**等。

**JDBC API可以访问任何RDBMS**，包含：

- JDBC Driver：驱动程序
- Connection：数据库连接
- Statement：执行SQL增删查改
- ResultSet：查询返回的结果集

### JDBC编程的步骤

1. 加载驱动（.jar包）
2. 连接数据库Connection
3. 创建Statement对象
4. 构造SQL DML语句
5. 适用Statement对象发送SQL
6. DB Server收到以后执行，返回ResultSet后，Client执行逻辑处理
7. 执行相关异常处理
8. 关闭Statement、Connection

### JDBC SQL执行调用方式

- execute()：所有的DDL、DML通用
- executeQuery()：SELECT
- executeUpdate()：INSERT、DELETE、UPDATE

### JDBC Prepared Statement语法

先设置statement的大致语句，然后补充细节：

``` java
PreparedStatement pStmt = conn.prepareStatement(
	"insert into instructor values(?,?,?,?)");
pStmt.setString(1, "88877");
pStmt.setString(2, "Perry");
pStmt.setString(3, "Finance");
pStmt.setInt(4, 125000);
pStmt.executeUpdate();
pStmt.setString(1, "88878");
pStmt.executeUpdate();
```

## SQL注入

原因：

![image-20221217215732148](/images/DBSNote/image-20221217215732148.png)

### SQL注入测试

可以使用经典的单引号判断法：`http://xxx/abc.php?id=1'`给服务器发送"1'"这个字符串，如果页面返回错误，那么一定有SQL注入存在。

### SQL注入的例子（JAVA）

#### 单引号'

假设JDBC编程时这样构造查询语句的：

``` java
"select * deom instructor where name ='" + name + "'"
```

如果发送`?name =X'or'Y'='Y` 

那么服务器端得到的语句就变成了：

``` sql
select * from instructor where name = 'X' or 'Y' = 'Y'
```

这样一来判断条件就变成了永真

#### SQL注释符--

假设程序逻辑：

``` java
"select * from users where username = '" + formusr + "'and password ='" + formpwd + "'";
```

如果输入的user是`admin'--`，password随便填（比如123），语句则为：

``` sql
select * from users where username = 'admin'--'and password = '123'
```

那么`--`后面的都会被注释掉，直接得到了admin

#### 分号;

程序：

``` java
"select * from users where userid = " + userid;
```

输入userid为`1;delete users;`：

``` sql
select * from users where userid = 1;delete users;
```

#### SQL注释符#

同--

#### 常用SQL注入特殊字符总结

![image-20221217221644607](/images/DBSNote/image-20221217221644607.png)

### SQL注入的预防

- 使用之前提到的Prepared Statement，必须用参数化的传递方式setXXX()
- 使用存储过程
- 根本方法：检查用户输入，转义特殊字符

## 小结

这一章内容比较少，一是JDBC、二是SQL注入及其防范

JDBC部分，介绍了其基本组件、使用JDBC操作数据库的基本流程、JDBC的三种调用函数、Prepared Statement

SQL注入方面，介绍了通过'、--、;、#等字符的注入构造，同时介绍了对于SQL注入的预防方法











