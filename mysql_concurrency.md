---
title: MySQL_并发控制
categories: 技术
tags: [mysql]
description:  mysql并发控制，包括锁、事务、隔离、死锁等概念的介绍，主要参考《MySQL 高性能优化》

---

## 读写锁
读锁：共享锁，
写锁：排他锁，阻止其他的读锁和写锁行为

## 锁粒度
在锁机制存在的情况下，提高共享资源并发性的方法是让锁定对象更准确。尽量只锁定需要修改的数据部分。理想的情况下，精确锁定会修改的数据片段。
另一方面，加锁也需要消耗资源。锁的各种操作，包括获得锁、检查锁是否已解除、释放锁等，都会增加系统开销。如果锁的操作比较频繁，系统会花大量的时间来管理锁，如不是执行数据存储，则兄台那个的性能也会受到影响。
可见，锁管理的实现对性能指标存在着相互冲突的正反影响，将锁粒度固定在某个级别，可以实现锁管理的灵活性、系统消耗两方面的综合考虑。表锁、行级锁是比较常见、重要的锁策略。

- 表锁
用户在对表进行写操作前，需获得写锁，如果是写锁是表锁，则会阻塞其他用户对该表的所有读写操作。
尽管存储引擎可以管理自己的锁，MySQL本身还是会使用各种有效的表锁来实现不同的目的。例如ALTER TABLE语句的执行会触发表锁，而忽略存储引擎的锁机制。

- 行级锁

## 事务
一组原子性的操作。ACID概念。

- 原子性(atomicity)：最小工作单元，要么都成功，要么都失败。
- 一致性(consistency)：从一个一致性的状态转到另外一个一致性的状态，从原子性理解，事务的操作不会部分成功，部分失败，因此不会对系统数据造成不一致
- 隔离性(isolation)：通常来说，事务在最终提交前，对其他事务是不可见的。事务执行的部分对数据的变更对其他事务屏蔽。
- 持久性(durability)：一旦事务提交，所执行的修改会永久的保存到数据库中

## 隔离级别
SQL标准中定义了4中隔离级别，低级别的隔离可以执行更高的并发，系统的开销也更低。

- READ UNCOMMITTED（未提交读）该隔离级别下，事务中的修改，即使没有提交，对其他事务也都是可见的。因此，对于其他业务，可能会产生“**脏读**”（读取到部分修改的、事务未提交的数据），从而引起很多问题。同时从性能层面考虑，
- READ UNCOMMITED 和其他隔离级别也差不多，因此实际场景中一般很少使用。脏读：事务T1更新了一行记录的内容，但是并没有提交所做的修改。事务T2读取更新后的行，然后T1执行回滚操作，取消了刚才所做的修改。现在T2所读取的行就无效了
- READ COMMITTED（提交读）大多数据库的默认隔离级别，但MySQL不是。本隔离级别下，满足隔离的基本定义：事务在提交前所做的修改对其他业务不可见。该级别下，两次执行同样的查询，可能会得到不一样的结果，产生不可重复读的效果。**不可重复读**：事务T1读取一行记录，紧接着事务T2修改 了T1刚才读取的那一行记录。然后T1又再次读取这行记录，发现与刚才读取的结果不同。这就称为“不可重复”读，因为T1原来读取的那行记录已经发生了变化。
- REPEATABLE READ（可重复读）MySQL默认隔离级别。该隔离级别解决了不可重复读问题，但还是存在**幻读**。InnoDB 通过多版本并发控制（MVCC）解决幻读问题。幻读：事务T1读取一条指定的WHERE子句所返回的结果集。然后事务T2新插入 一行记录，这行记录恰好可以满足T1所使用的查询条件中的WHERE 子句的条件。然后T1又使用相同的查询再次对表进行检索，但是此时却看到了事务T2刚才插入的新行。这个新行就称为“幻像”，因为对T1来说这一行就像突然出现的一样。
SERIALIZE（可串行化）它强制事务串行执行。该隔离级别下，会对读取的每一行数据上都加上锁。因而对锁机制的管理比较耗系统资源。

| 隔离级别 | 脏读可能性 | 不可重复读可能性 | 幻读可能性 | 加锁读 | 
|--------|----------|---------------|--------|--------|
|READ UNCOMMITTED |          是|        是|        是|        否|
|READ COMMITTED	|        否|        是|        是|        否|
|REPEATABLE READ|        否|        否|        是|        否|
|SERIALIZE	|        否|        否|        否|        是|

## 死锁
### 在事务中混合使用存储引擎
MySQL服务器层不管理事务，事务是由存储引擎实现的。所以在同一个事务中，使用多个存储引擎是不可靠的。

具体为：在正常提交情况下不会有问题，但如果事务需要回滚，非事务型的表变更会无法撤销，从而导致数据库表数据不一致。
### 多版本并发控制
MVCC, 前面有提到。MySQL的大多数事务型存储引擎的实现都不是简单的行级锁。

MVCC 的实现，是通过保存数据在某个时间点的快照实现的。即不管需要执行多长时间，每个事务看到的数据都是一致的。根据事务开始的时间不同，每个事务对同一张表，同一时刻看到的数据可能是不一样的。

InnoDb 的 MVCC，是通过在每行记录后面保存两个隐藏的列来实现的。这两个列，一个保存了创建时间，一个保存行的过期时间（或删除时间）。当然存储的并不是实际的时间值，而是系统版本号。每开始一个新的事务，系统版本号都会自动增加。

## 锁的说明

### 行级锁
行级锁是针对索引生效的，并非存储数据。每个B+Tree上的行锁，也可以理解为每个Index上的行锁。

- ==操作一行记录时，有可能会加多个行锁在不同的B+Tree上。==
- ==只有通过索引条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁！==

以下实例参考：http://book.51cto.com/art/200803/68127.htm
1） 在不通过索引条件查询的时候，InnoDB确实使用的是表锁，而不是行锁。

```bash
mysql> create table tab_no_index(id int,name varchar(10)) engine=innodb;
Query OK, 0 rows affected (0.15 sec)
mysql> insert into tab_no_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
Query OK, 4 rows affected (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 0
```

如在上表执行

```
select * from tab_no_index where id = 1 for update; 
```
会产生表锁。


2） 创建`tab_with_index`表，id字段有普通索引

```bash
mysql> create table tab_with_index(id int,name varchar(10)) engine=innodb;
Query OK, 0 rows affected (0.15 sec)
mysql> alter table tab_with_index add index id(id);
Query OK, 4 rows affected (0.24 sec)
Records: 4  Duplicates: 0  Warnings: 0
```

在上表执行

```sql
select * from tab_no_index where id = 1 for update; 
```
产生行锁。

3） 由于MySQL的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用相同的索引键，是会出现锁冲突的。应用设计的时候要注意这一点。

```bash
mysql> select * from tab_with_index where id = 1 and name = '1' for update;
```

```bash
mysql> select * from tab_with_index where id = 1 and name = '4' for update;
```
上述两条 sql 虽然不是操作同一条记录，但是由于 index(id=1)产生了锁，因此还是会出现锁等待情况。

4） 当表有多个索引的时候，不同的事务可以使用不同的索引锁定不同的行，另外，不论是使用主键索引、唯一索引或普通索引，InnoDB都会使用行锁来对数据加锁。可以理解为所有索引树都会加锁。

```bash
mysql> alter table tab_with_index add index name(name);
Query OK, 5 rows affected (0.23 sec)
Records: 5  Duplicates: 0  Warnings: 0
```

```bash
mysql> select * from tab_with_index where id = 1 for update;
+------+------+
| id   | name |
+------+------+
| 1    | 1    |
| 1    | 4    |
+------+------+
2 rows in set (0.00 sec)
```

```bash
#Session_2使用name的索引访问记录，因为记录没有被索引，所以可以获得锁
mysql> select * from tab_with_index where name = '2' for update;
+------+------+
| id   | name |
+------+------+
| 2    | 2    |
+------+------+
1 row in set (0.00 sec)
#由于访问的记录已经被session_1锁定，所以等待获得锁
mysql> select * from tab_with_index where name = '4' for update;
```

5）即便在条件中使用了索引字段，但是否使用索引来检索数据是由MySQL通过判断不同执行计划的代价来决定的，如果MySQL认为全表扫描效率更高，比如对一些很小的表，它就不会使用索引，这种情况下InnoDB将使用表锁，而不是行锁。因此，在分析锁冲突时，别忘了检查SQL的执行计划，以确认是否真正使用了索引。

6）检索值的数据类型与索引字段不同，虽然MySQL能够进行数据类型转换，但却不会使用索引，从而导致InnoDB使用表锁。

### select for update
mysql中使用select for update的必须针对InnoDb，并且是在事务中，才能起作用。select的条件不一样，采用的是行级锁还是表级锁也不一样。由于 InnoDB 预设是 Row-Level Lock，所以只有「明确」的指定主键，MySQL 才会执行 Row lock (只锁住被选取的资料例) ，否则 MySQL 将会执行 Table Lock (将整个资料表单给锁住)。举个例子:假设有个表单 products ，裡面有 id 跟 name 二个栏位，id 是主键。- 例1 (明确指定主键，并且有此条记录，row lock):SELECT * FROM products WHERE id='3' FOR UPDATE;- 例2 (明确指定主键，若查无此条记录，无 lock):SELECT * FROM products WHERE id='-1' FOR UPDATE;- 例3 (无主键，table lock):SELECT * FROM products WHERE name='Mouse' FOR UPDATE;- 例4 (主键不明确，table lock):SELECT * FROM products WHERE id<>'3' FOR UPDATE;- 例5 (主键不明确，table lock):SELECT * FROM products WHERE id LIKE '3' FOR UPDATE;

