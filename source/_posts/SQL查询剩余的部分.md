---
title: SQL查询剩余的部分
date: 2022-10-07 19:56:26
categories: 数据库原理
tags: [数据库原理, mysql]
---

写在上一篇：SQL连接之后，继续补全SQL查询的内容和demo

## 嵌套查询（子查询）

也就是把一个查询块潜逃到另一个查询快的WHERE或者HAVING子句中，一个最简单的例子：

``` mysql
mysql> SELECT Sname FROM Student WHERE Sno IN (SELECT Sno FROM SC WHERE Cno='2');

+-------+
| Sname |
+-------+
| 李勇  |
| 刘晨  |
+-------+
2 rows in set (0.02 sec)
```

这个命令的意思非常简单，就是先查询SC表中Cno='2'的Sno，然后在student表中查询这些Sno对应的Sname。

### 嵌套查询的求解方法

#### 不相关情况

对于上面这种例子中，子查询条件不依赖父查询的情况，从内到外依次执行两次查询即可

#### 相关情况

也就是说子句的查询条件对父查询是有依赖的，具体例子如下：

``` sql
mysql> SELECT Sno,Cno FROM sc x WHERE Grade >= (SELECT AVG(Grade) FROM sc y WHERE y.Sno = x.Sno);
+-----------+-----+
| Sno       | Cno |
+-----------+-----+
| 201215121 | 1   |
| 201215122 | 2   |
+-----------+-----+
2 rows in set (0.00 sec)
```

这个嵌套查询得到的结果是：查找某个学生分数高于该学生**自己平均分**的课程，子句中用到了父查询的属性x.Sno，所以自然不能和上面一样从内到外执行。

那么就只能老老实实从外到内执行，通过遍历外层元组的方式查询。

### 嵌套查询的种类

#### 带有IN子句的嵌套查询

``` sql
mysql> SELECT Sno,Sname,Sdept FROM Student WHERE Sdept IN (SELECT Sdept FROM Student WHERE Sname = '刘晨');
+-----------+-------+-------+
| Sno       | Sname | Sdept |
+-----------+-------+-------+
| 201215121 | 李勇  | CS    |
| 201215122 | 刘晨  | CS    |
+-----------+-------+-------+
2 rows in set (0.00 sec)
```

首先IN子句内查询的是刘晨所在的系，外层查询的意思就是查找刘晨所在的系（即CS系）的学生。

这个语句照样可以用之前学到的自身连接完成查询：

``` sql
mysql> SELECT S1.Sno,S1.Sname,S1.Sdept FROM Student S1, Student S2 WHERE S1.Sdept = S2.Sdept AND S2.Sname = '刘晨';
+-----------+-------+-------+
| Sno       | Sname | Sdept |
+-----------+-------+-------+
| 201215121 | 李勇  | CS    |
| 201215122 | 刘晨  | CS    |
+-----------+-------+-------+
2 rows in set (0.00 sec)
```

**另一个例子**：查询选修课程名为”信息系统“的学生的学号和姓名

思路：学生的学号和姓名在student表中，同时学号也在sc表中，sc表中包含cno但是没有cname

所以我们先在course中查询cname=“信息系统”的cno，然后在sc中查询cno符合要求的sno，然后在student表中查找这些sno对应的sname

用两层IN实现上述逻辑就可以了

``` sql
mysql> SELECT Sno,Sname FROM Student WHERE Sno IN (SELECT Sno FROM sc WHERE Cno IN (SELECT Cno FROM course WHERE Cname='信息系统') );
+-----------+-------+
| Sno       | Sname |
+-----------+-------+
| 201215121 | 李勇  |
| 201215122 | 刘晨  |
+-----------+-------+
2 rows in set (0.00 sec)
```

当然和上面一样我们也可以使用等值连接来查询，将三个表连接到一块再查询Cname就好了，当然也可以course这个表先查询后连接

#### 带比较运算符的嵌套查询

由于比较运算符是元组之间的运算，而不是元组集合间的运算，所以需要注意**子查询返回的结果只有一个元组**，否则会报错。

例如上面的例子1中，子查询返回的结果就只有一个元组，所以可以简单地把IN换成=

``` sql
mysql> SELECT Sno,Sname,Sdept FROM Student WHERE Sdept = (SELECT Sdept FROM Student WHERE Sname = '刘晨');
+-----------+-------+-------+
| Sno       | Sname | Sdept |
+-----------+-------+-------+
| 201215121 | 李勇  | CS    |
| 201215122 | 刘晨  | CS    |
+-----------+-------+-------+
2 rows in set (0.00 sec)
```

#### 带ANY(SOME)或ALL的嵌套查询

这些谓词常常与比较运算符对比使用，表示与子查询中的任意或者所有复合要求

这些谓词可以用来代替聚类函数，如

WHERE Sage < ANY(SELECT Sage FROM ... WHERE ...) 和

WHERE Sage < (SELECT MAX(Sage) FROM ... WHERE ...) 就是等价的

注意有的时候千万别脑补想当然了，比如说我最开始看到<ANY的时候就想到：小于任何？那不就是<MIN(Sage)吗！

但是实际上这个<ANY的意识是只需要存在一个或以上比它大的就可以了。所以我觉得用SOME是不是比ANY看起来更舒服呢...

#### 带有EXISTS的查询

即存在$\exists$，EXISTS子句只返回True或者False：子查询的结果为空返回FALSE、否则返回TRUE。

**e.g.1** 查选修课Cno为1的课程的学生姓名

``` sql
mysql> SELECT Sname FROM Student WHERE EXISTS (SELECT * FROM SC WHERE Sno=Student.Sno AND Cno='1');
+-------+
| Sname |
+-------+
| 李勇  |
+-------+
1 row in set (0.00 sec)
```

一般来说，由于EXISTS只看元组个数，所以SELECT *就可以了，当然非要SELECT+某些属性也不是不行，反正不影响判断逻辑

##### NOT EXISTS与关系÷运算

先从一个例子入手：查询选修了全部课程的学生姓名

查询姓名先放到一边，这个表主要是在SC中查询对应Cno包含了所有course中Cno的Sno，那这不就是关系除，然后再和Student表连接投影其Sname，关系代数为：

$$\pi_{Sname}(Student \bowtie (\pi_{Sno,Cno}(SC) \div \pi _{Cno}(Course))$$

最关键的就是这个关系除了，表示一个任意$\forall$的关系，但是SQL语言没有这个关系，所以我们只能用等式$(\forall x)P \equiv \urcorner (\exists x) (\urcorner P)$。

具体这个例子而言，也就是**不存在**一门课，使得这个学生**没有选**它。这个双重否定句就是结题的关键，我们可以轻松使用两个NOT EXISTS表示这句话。

翻译成SQL：

``` sql
mysql> SELECT Sname FROM Student WHERE NOT EXISTS (SELECT * FROM Course WHERE NOT EXISTS (SELECT * FROM sc WHERE Sno = Student.Sno AND Cno = Course.Cno) );
```

当然对于我们的数据库demo而言，没有这么一个肝帝把课全选了，所以就返回Empty了。

当然 对于这种特殊问题，我们甚至可以使用COUNT聚集函数。只要这个学生的COUNT(Cno)>=Course所有的COUNT(Cno)就可以了。

``` sql
mysql> SELECT Sname FROM Student WHERE Sno IN (SELECT Sno FROM sc GROUP BY Sno HAVING COUNT(cno) >= (SELECT COUNT(DISTINCT Cno) FROM course));
Empty set (0.00 sec)
```

**有关÷的第二个例子**

查询至少选修了学生201215122所选全部课程的学生学号

其实就是把$$\pi_{Sname}(Student \bowtie (\pi_{Sno,Cno}(SC) \div \pi _{Cno}(Course))$$除运算右边的集合换成201215122所选的Cno的集合，还是用两个NOT EXISTS解题。

``` sql
mysql> SELECT sno, sname FROM student WHERE NOT EXISTS (SELECT * FROM (SELECT cno FROM sc WHERE sc.sno = '201215122') c WHERE NOT EXISTS (SELECT * FROM sc WHERE sc.sno = student.sno AND c.Cno = sc.Cno));
+-----------+-------+
| sno       | sname |
+-----------+-------+
| 201215122 | 刘晨  |
| 201215121 | 李勇  |
+-----------+-------+
2 rows in set (0.00 sec)
```

但是注意下面使用COUNT比较的方法就不能用了

不过SQL的查询表达真的是很多样化，这里再给出2种解法（其中第二种MySQL不支持，只能用Oracle）：

**解法1**：外连接

左表：201215122选的所有课程

右表：需要判断的学生选的课程

外连接以后，对于左表中的某一门课程，如果这个学生没选，右表会出现NULL。我们只要保证这个学生NOT EXISTS连接以后的右表有NULL就好了

``` sql
mysql> SELECT Sno,Sname FROM student WHERE NOT EXISTS ( SELECT * FROM (SELECT * FROM course NATURAL JOIN sc WHERE sc.sno = '201215122') a LEFT JOIN (SELECT * FROM course NATURAL JOIN sc WHERE sc.sno = student.sno) b ON a.cno = b.cno WHERE b.cno IS NULL );
+-----------+-------+
| Sno       | Sname |
+-----------+-------+
| 201215122 | 刘晨  |
| 201215121 | 李勇  |
+-----------+-------+
2 rows in set (0.00 sec)
```

**解法2**：使用集合差，不过MySQL不支持这个操作，只有Oracle可以用

``` sql
SELECT Sno FROM Student
	WHERE NOT EXISTS
	(
    SELECT Cno FROM Course NATURAL JOIN sc WHERE sc.sno = '201215122' --指定学生选的所有课
        MINUS
    SELECT Cno FROM Course NATURAL JOIN sc WHERE sc.sno = student.sno  --要判断的学生选的所有课   
    );
```

## 集合查询

集合查询主要有UNION、INTERSECT、EXCEPT/MINUS三个，其中MySQL只支持UNION操作

## 基于派生表的查询

派生表也可以看作是一种嵌套查询，只是把子查询的位置放到了FROM子句中。

其有三种不同的语法

1. 派生表计算式 AS 派生表名(派生表列名)
2. 派生表计算式 派生表名
3. WITH 派生表名 AS 派生表计算式 + SELECT父查询

注意，如果派生表里面使用了聚集函数，那么一定要定义列名，因为聚集函数结果列本身是没有名字的。但要是没有派生函数的话，就可以直接用 派生表名.列名 表示其中的列。
