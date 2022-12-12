---
title:  数据库原理第三章 SQL
date: 2022-12-10 20:25:42
categories: 数据库系统原理
tags: [数据库]

---

## SQL概述

### SQL的产生和发展

SQL在IBM诞生，起初由Codd提出关系数据建模，而后Chamberlin和Boyce开发了SEQUEL语言，后续改名SQL

### SQL的特点

- 综合统一：包含数据定义语言DDL、数据操纵语言DML、数据控制语言DCL、事务控制语言TCL（Transaction Control Language）。SQL可以独立完成数据库生命周期的全部活动。
- 高度非过程化：存取路径的选择由DBMS自动完成
- 面向集合操作：这一点**与关系操作一样**，SQL操作的对象和结果都是集合
- 提供多种使用方式：除了这两章介绍的SQL以外，SQL还可以使用ODBC、JDBC等API被高级语言使用
- 简介：语法核心功能只用了9个动词

### SQL的基本概念

SQL也是支持数据库的三级模式结构的，SQL从内到外为：存储文件、基本表、视图。三者间的关系以及与各级模式的对应关系如下：

![image-20221209101513396](/images/DBSNote/image-20221209101513396.png)

**视图**就是用户可以看到的基本表的部分，所以是外模式：要注意的是SQL的**一个视图可以是由多个表导出**的

**基本表**自然而然就是全局的逻辑数据，构成模式；**存储文件**构成内模式，要注意**基本表和存储文件**之间的关系可以是**一对一、多对一**。

## 数据定义

SQL可以定义模式、表、视图与索引，主要用到CREATE、DROP、ALTER三个关键字，具体如下：

![image-20221209103131184](/images/DBSNote/image-20221209103131184.png)

注意：模式和视图是**不能修改**的

RDBMS的**层次化对象命名**机制如下图：

![image-20221209103521068](/images/DBSNote/image-20221209103521068.png)

其中，**一个数据库实例可以有多个模式、一个模式下通常有多个表**。

### 定义模式

创建：

``` sql
CREATE SCHEMA <模式名> AUTHORIZATION <用户名>;
```

模式名省略的时候，自动设置为用户名

定义模式相当于定义了一个**命名空间**

删除：

``` sql
DROP SCHEMA <模式名> <CASCADE|RESTRICT>;
```

CASCADE级联表示删除模式下的所有对象；RESTRICT限制表示如果模式非空则不可以删除。

**关于不同数据库中模式的地位：**MySQL中Schema和数据库是等价概念，Oracle中的Schema是User名下所有数据库对象的总和，和用户名相同。简单来说：**MySQL中Schema=Database，Oracle中Schema=user**。

### 定义基本表

创建：

``` sql
CREATE TABLE <表名>
      (<列名> <数据类型>[ <列级完整性约束条件> ]
      [,<列名> <数据类型>[ <列级完整性约束条件>] ] 
		...
      [,<表级完整性约束条件> ] );
```

定义表所属的模式：

- 在创建表时使用 <模式名>.<表名>
- 在创建模式时同时创建表
- 设置表所属的模式
- 倘若没有指定，系统会根据**搜索路径**来确定其模式

修改：

``` sql
ALTER TABLE <表名>
[ADD [COLUMN] <新列名> <数据类型> [完整性约束]]
[ADD <表级完整性约束>]
[DROP [COLUMN] <列名> [CASCADE| RESTRICT]]
[DROP CONSTRAINT <完整性约束名> [RESTRICT|CASCADE]]
[ALTER COLUMN <列名> <数据类型>];
```

也就也是说，对于一个基本表，可以加、减、修改列，设置和取消完整性约束。

删除：

``` sql
DROP TABLE <表名> [RESTRICT|CASCADE];
```

同样的有级联和限制之分：设计到其他表的引用，如Foreign Key、View等

### 定义索引

常见的索引有：顺序、B+树、hash、bitmap

**DBA或者表的属主**可以建立索引，索引由DBMS自动维护。**索引不能被用户显式选择**，系统会自动判断是否有存在的索引可以加速性能。

创建：

``` sql
CREATE [UNIQUE] [CLUSTER] INDEX <索引名> ON <表名>(<列名>[<次序>], [<列名>[<次序>]])：
```

其中UNIQUE表示一个索引值只对应一条记录，CLUSTER表示建立聚簇索引。次序为ASC升序、DESC降序。

重命名：

``` SQL
ALTER INDEX <旧名> RENAME TO <新名>;
```

删除：

``` sql
DROP INDEX <索引名>;
```

### 数据字典

数据字典是一张DBMS内部的表，记录了数据库的所有定义信息：

- 关系模式定义
- 视图定义
- 索引定义
- 完整性约束定义
- 用户权限
- 统计信息等

## 数据查询

``` sql
SELECT [ALL|DISTINCT] <目标列表达式>[,<目标列表达式>] …
FROM <表名或视图名>[,<表名或视图名> ]…|(SELECT 语句)      
	[AS] <别名>
[WHERE <条件表达式>]
[GROUP BY <列名1> [HAVING <条件表达式>]]
[ORDER BY <列名2> [ASC|DESC]];
```

### 单表查询

ALL|DISTINCT表是否保留重复

目标列可以有别名，也可以使用表达式作为目标列

除了运算符以外，表达式还可以使用IN、LIKE、BETWEEM...AND...等

ORDER BY若没有标明ASC/DESC则升序排列，对于NULL的排序，由具体系统决定

GROUP BY可以细化聚集函数的作用对象，如果要对聚集函数进行筛选，不**能够将聚集函数放在WHERE中，需要放在GROUP BY后面的HAVING子句中！！！**

### 连接查询

这一块的表达非常多样化，例如条件，最基本的是写在WHERE子句中、但其实也可以写在ON子句中，而且两者是有区别的：WHERE可以看作先做笛卡尔积后筛选、ON可以看作使用nested loop join一边join一边筛选，所以在外连接一些情况下面，两者的结果会有所不同。如果是自然连接的话，还可以使用`JOIN...USING`表达自然连接的列。

连接查询这部分在此前以及有过内容了，可以参阅：[SQL连接 | Boulevard's Blog (blog-boulevard.top)](https://blog-boulevard.top/2022/10/02/sql连接/#自身连接)

### 子查询、集合查询、派生表查询等

参见[SQL查询剩余的部分 | Boulevard's Blog (blog-boulevard.top)](https://blog-boulevard.top/2022/10/07/sql查询剩余的部分/)，子查询主要有IN、比较运算符、ANY(SOME)/ALL、EXISTS等方法，特别注意双NOT EXISTS表示关系除。

### SQL的特别用法

- CASE语句：

  ``` sql
  CASE
  	WHEN Condition then result
  	...
  	ELSE result
  END;
  ```

- 返回结果的前N条记录：

  - SQL Server： `SELECT TOP N|百分比`
  - MySQL：`SELECT FROM WHERE... LIMIT N`
  - Oracle：`SELECT FROM WHERE ROWNUM<=N`

## 数据更新

### 插入数据

可以插入单条：

``` sql
INSERT INTO <表名> [<属性列>...]
VALUES (<值>...);
```

也可以插入子查询结果：

``` sql
INSERT INTO <表名> [<属性列>...]
子查询;
```

插入之前会检查三个完整性

### 复制表

#### 复制结构

``` sql
CREATE TABLE new LIKE orig;
```

#### 复制结构和数据

``` sql
CREATE TABLE new [AS] SELECT * FROM orig;
```

#### 先复制结构后复制数据

``` sql
CREATE TABLE new LIKE orig;
INSERT INTO new SELECT * FROM orig;
```

复制过程中，需要注意：

- 主、外键、索引是否被复制（通常没有）
- 约束是否被复制

在复制过程中，会禁止索引、外键、触发器等

### 修改数据

``` sql
UPDATE <表名>
SET <列>=<>,...
[WHERE <条件>];
```

修改数据的时候，也需要临时禁止约束（外键、check、index、trigger等）

### 删除数据

``` sql
DELETE
FROM <表>
[WHERE <条件>];
```

## 空值的处理

空值就是“不确定”、“不存在”的意思，只能用IS [NOT] NULL判断，而不能用=NULL。

如果有NOT NULL、UNIQUE或者是作为码属性则不允许为空

## 视图

从一个**或者多个**基本表**或视图**导出的虚表

在数据库中，视图只有定义，不存放数据

### 定义视图

``` sql
CREATE VIEW
	<视图名> [<列名>,...]
AS <子查询>
[WITH CHECK OPTION];
```

所谓WITH CHECK OPTION就是说更改视图的时候，需要保证满足视图定义的谓词条件（也就是满足子查询的条件表达式）

比如说，基本表是所有学生Student，给1系创建一个视图Student_Dept1，那么在开启了WITH CHECK OPTION的情况下，这个系的老师若插入一条学生纪录、系号为2，则会报错。

### 删除视图

``` SQL
DROP VIEW <视图名> [CASCADE];
```

之前说过，视图是可以从其他视图导出的，开启级联以后，如果该视图导出了其他视图，就会把导出的其他视图也删掉。

### 查询视图

对于用户来说查询视图和查询基本表没有什么区别

对DBMS来说，常用视图消解法实现对于视图的查询：转化为对于基本表的等价查询

试图消解法有时候不能产生正确的查询，具体可以看下面的例子：

定义S_G视图：

``` sql
CREATE VIEW S_G(Sno,Gavg)
AS
SELECT Sno,AVG(Grade)
FROM SC
GROUP BY Sno;
```

对其查询：

``` SQL
SELECT * FROM S_G
WHERE Gavg>=80l;
```

视图消解以后，就变成了：

``` sql
SELECT Sno,AVG(Grade) FROM SC
WHERE AVG(Grade)>=80
GROUP BY Sno;
```

而之前提过，聚集函数不能够出现在WHERE中，而要使用HAVING函数

### 更新视图

同样的，也是使用视图消解法

但是，还是同样的，视图的更新也会有一些问题：

比如上面的S_G就不能更新，以外Gavg是聚合函数产生的列，是无法转化到基本表的

一般来说有如下情况的不能更新：

- 使用了聚合函数
- 使用了UNIQUE、DISTINCT
- 使用的GROUP BY
- 使用了算术表达式
- 视图没有包括基本表的主键
- ...

一般来说，只允许给一个表的行列子集构成的视图更新

### 视图的作用

- 简化操作
- 从多个角度看同一数据
- 重构数据库的逻辑独立性
- 保护机密数据
- 更清晰的表达查询

## 小结

这一章的东西真的很多，除了SQL的一些历史、特点、概念以外，主要介绍了SQL对于数据的定义、查询、更新、以及视图

对于数据定义，要清楚模式和视图是不能够更改(ALTER)的。所有的数据定义被存在了数据字典中。

数据的查询这部分比较复杂：

对于普通的查询而言，要注意聚合函数与GROUP BY ... HAVING ...

对于连接查询，要注意用USING、WHERE、ON来连接的区别

对于子查询而言，有IN、ANY(SOME)/ALL、EXISTS等方法，要注意两个NOT EXISTS表示关系除的方法

同时，查询的时候还可以使用CASE语句、控制结果返回长度等。

对于数据的更新，这部分比较简单

对于视图，主要要注意视图消解法，以及对于视图查询和更新的限制
