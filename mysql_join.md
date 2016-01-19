---
title: MySQL_JOIN
categories: 技术
tags: [mysql]
description:  mysql join, 介绍各种连接的概念与区别

---

## 连接的概念
连接分为条件连接、等值连接和自然连接三种。
- 条件连接就是在多个表的笛卡尔积中选取满足条件的行的连接，例如  select * from A,B where A.a > A.b  之类的有条件的查询。
- 等值连接就是特殊的条件连接，当条件为某字段=某字段时，即为等值连接。如SELECT ename,sal,dname FROM emp,dept WHERE emp.deptno=dept.deptno;
- 自然连接是一种特殊的等值连接，他要求多个表有相同的属性字段，然后条件为相同的属性字段值相等，最后再将表中重复的属性字段去掉，即为自然连接。如A中a,b,c字段，B中有c,d字段，则select * from A natural join B  相当于 select A.a,A.b,A.c,B.d from A.c = B.c  。

Join操作基本分为3大类：外连接(细分为：左连接、右连接、全连接)、自然连接、内连接
Join操作的共性：第一步均为将所有参与操作的表进行了一个笛卡儿积，然后才依据各连接条件进行记录的筛选

## 内连接与等值连接的区别：
内连接：两个表（或连接）中某一数据项相等的连接称为内连接。等值连接一般用where字句设置条件，内连接一般用on字句设置条件，但内连接与等值连接效果是相同的。
内连接与等值连接其实是一回事情（等效）。
经常有人会问到select a.id,b.name from a,b where a.id=b.pid  与
select a.id,b.name from a inner join b on a.id=b.pid有什么区别，哪个效率更高一些。
实际上一回事情了。只是内连接是由SQL 1999规则定的书写方式。两个说的是一码事。

## 内连接与外连接的结果集区别：
假设我们有两张表。
Table A是左边的表。
Table B是右边的表。
其各有四条记录，其中有两条记录是相同的，如下所示：

|id | name  |   id  |  name|
|:----|:---|:----|:----|
|1 |Pirate        |1| Rutabaga|
|2 |Monkey    |2| Pirate|
|3 |Ninja         |3| Darth Vader|
|4 |Spaghetti  |4| Ninja|

下面让我们来看看不同的Join会产生什么样的结果。

## 不同的 join 结果

### Inner join  == A&B
产生的结果集中，是A和B的交集
SELECT * FROM TableA INNER JOIN TableB ON TableA.name = TableB.name

|id | name  |   id  |  name|
|:----|:---|:----|:----|
|1  |Pirate|    2   | Pirate|
|3  |Ninja |    4   | Ninja|
![](http://7xq67w.com1.z0.glb.clouddn.com/mysql%2Fmysql-join-and.png)

### Full outer join == A || B
SELECT * FROM TableA FULL OUTER JOIN TableB ON TableA.name = TableB.name

|id | name  |   id  |  name|
|:----|:---|:----|:----|
|1    |Pirate|    2|    Pirate|
|2    |Monkey| null| null|
|3    |Ninja   |   4|    Ninja|
|4    |Spaghetti| null| null|
|null |null |      1 |   Rutabaga|
|null |null |     3 |   Darth Vader|
产生A和B的并集。但是需要注意的是，对于没有匹配的记录，则会以null做为值。
![](http://7xq67w.com1.z0.glb.clouddn.com/mysql%2Fmysql-join-or.png)

### Left outer join = A
产生表A的完全集，而B表中匹配的则有值，没有匹配的则以null值取代。
SELECT * FROM TableA LEFT OUTER JOIN TableB ON TableA.name = TableB.name

|id | name  |   id  |  name|
|:----|:---|:----|:----|
|1  |Pirate  |  2  |  Pirate|
|2  |Monkey| null| null|
|3  |Ninja  |    4  |  Ninja|
|4  |Spaghetti| null |null|
![](http://7xq67w.com1.z0.glb.clouddn.com/mysql%2Fmysql-join-a-all.png)

### A-B
SELECT * FROM TableA LEFT OUTER JOIN TableB ON TableA.name = TableB.name WHERE TableB.id IS null

|id | name  |   id  |  name|
|:----|:---|:----|:----|
|2  |Monkey |null |null|
|4  |Spaghetti |null |null|
产生在A表中有而在B表中没有的集合。
![](http://7xq67w.com1.z0.glb.clouddn.com/mysql%2Fmysql-join-a-sub-b.png)

### A表和B表都没有出现的数据集
SELECT * FROM TableA FULL OUTER JOIN TableB ON TableA.name = TableB.name WHERE TableA.id IS null OR TableB.id IS null

|id | name  |   id  |  name|
|:----|:---|:----|:----|
|2    |Monkey |null |null|
|4    |Spaghetti |null |null|
|null |null       |1    |Rutabaga|
|null |null       |3    |Darth Vader|
产生A表和B表都没有出现的数据集。
![](http://7xq67w.com1.z0.glb.clouddn.com/mysql%2Fmysql-join-ab.png)

还需要注意的是我们还有一个是“交差集” cross join, 这种Join没有办法用文式图表示，因为其就是把表A和表B的数据进行一个N*M的组合，即笛卡尔积。表达式如下：
SELECT * FROM TableA **CROSS JOIN** TableB
这个笛卡尔乘积会产生 4 x 4 = 16 条记录，一般来说，我们很少用到这个语法。但是我们得小心，如果不是使用嵌套的select语句，一般系统都会产生笛卡尔乘积然再做过滤。这是对于性能来说是非常危险的，尤其是表很大的时候。
![](http://7xq67w.com1.z0.glb.clouddn.com/mysql%2Fmysql-join-total.jpg)