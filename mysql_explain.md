---
title: MySQL_explain 的使用
categories: 技术
tags: [mysql]
description:  介绍 mysql-explain 命令的使用

---


## EXPLAIN 的使用


以下说明运行表结构。
```sql
 CREATE TABLE `iknow_team_user` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id',
  `teamId` int(11) unsigned NOT NULL COMMENT 'teamId',
  `uid` int(11) unsigned NOT NULL COMMENT 'uid',
  `username` varchar(32) NOT NULL DEFAULT '' COMMENT 'username',
  `joinTime` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'joinTime`',
  `uType` smallint(8) unsigned NOT NULL DEFAULT '0',
  `tagList` varchar(512) NOT NULL DEFAULT '',
  `extPack` varchar(2000) DEFAULT '' COMMENT 'json',
  PRIMARY KEY (`id`),
  UNIQUE KEY `ituid_tid` (`uid`,`teamId`),
  KEY `ittid_utype` (`teamId`,`uType`)
) ENGINE=InnoDB AUTO_INCREMENT=452786 DEFAULT CHARSET=gbk COMMENT='team_user'
```
### 1. EXPLAIN tbl_name
EXPLAIN tbl_name 是 DESCRIBE tbl_name 或 SHOW COLUMNS FROM tbl_name 的一个同义词。
举例如下:
```sql
explain iknow_team_user;
+----------+----------------------+------+-----+---------+----------------+
| Fie      | Type                 | Null | Key | Default | Extra          |
+----------+----------------------+------+-----+---------+----------------+
| id       | int(11) unsigned     | NO   | PRI | NULL    | auto_increment |
| teamId   | int(11) unsigned     | NO   | MUL | NULL    |                |
| uid      | int(11) unsigned     | NO   | MUL | NULL    |                |
| username | varchar(32)          | NO   |     |         |                |
| joinTime | int(11) unsigned     | NO   |     | 0       |                |
| uType    | smallint(8) unsigned | NO   |     | 0       |                |
| tagList  | varchar(512)         | NO   |     |         |                |
| extPack  | varchar(2000)        | YES  |     |         |                |
+----------+----------------------+------+-----+---------+----------------+
8 rows in set (0.00 sec)
```
### 2. EXPLAIN [EXTENDED | PARTITIONS] SELECT select_options
当在一个SELECT语句前使用关键字EXPLAIN时,MYSQL会解释将如何运行该SELECT语句,它显示了表如何连接、连接的顺序等信息。
表以它们在处理查询过程中将被MySQL读入的顺序被列出,而不表示执行顺序,每行显示的是执行计划的每一个组成部分以及执行的次序,查询里的每张表对应输出结果集中的一行, 这里表的比较广泛,子查询、union结果集。。。也在此范围内。
举例如下:
```sql
explain select * from iknow_team_user where id<100;
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+-------------+
| id | select_type | table           | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+-------------+
|  1 | SIMPLE      | iknow_team_user | range | PRIMARY       | PRIMARY | 4       | NULL |   73 | Using where |
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+-------------+
1 row in set (0.01 sec)
```
#### 2.1 EXPLAIN EXTENDED
5.1 之后增加了一个额外的过滤列extended,它告知服务器把执行计划反编译成select语句, 可以通过show warnings看到这些生成的语句,通过查看这些语句可以知道查询优化器怎么转化查询语句。
```sql
explain extended select * from iknow_team_user where id<100;
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table           | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | iknow_team_user | range | PRIMARY       | PRIMARY | 4       | NULL |   73 |   100.00 | Using where | 
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```
```sql
show warnings\G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: select `test`.`iknow_team_user`.`id` AS `id`,`test`.`iknow_team_user`.`teamId` AS `teamId`,`test`.`iknow_team_user`.`uid` AS `uid`,`test`.`iknow_team_user`.`username` AS `username`,`test`.`iknow_team_user`.`joinTime` AS `joinTime`,`test`.`iknow_team_user`.`uType` AS `uType`,`test`.`iknow_team_user`.`tagList` AS `tagList`,`test`.`iknow_team_user`.`extPack` AS `extPack` from `test`.`iknow_team_user` where (`test`.`iknow_team_user`.`id` < 100)
1 row in set (0.01 sec)
```
#### 2.2 EXPLAIN PARTITIONS
如果使用了mysql的分区表,explain partitions 显示查询需要访问的数据分片信息。
### 3. 重写非select查询
explain只能解释select查询,无法解释存储过程、insert、delete、update等其他类似的语句,可以把非select语句转化为对等的 select 访问请求。
### 4. EXPLAIN 使用注意点
#### 4.1 并非所有的 explain 后面的 select 语句都不会执行
from 子句中包含的子查询会执行,mysql 会执行这个子查询,并把结果集放入一个临时表中, 故谨慎在线上执行此类 explain 语句。
#### 4.2 explain 只是一个近似
explain 只是一个近似,它大部分时候是一个很好的近似,但是有时候也会远离实际情况,一些显示出来的统计信息都是估算的,不精确。
## EXPLAIN 中的列
首先进行如下约定:
简单查询 SIMPLE
复杂查询:简单子查询 SUBQUERY、衍生表&lt;derived2&gt;、union
### 1. id 列
表以它们在处理查询过程中将被 MySQL 读入的顺序被列出,而不表示执行顺序。查询里的 每张表对应输出结果集中的一行,使用 id 来标示出属于哪一行,id 大小顺序并不一定跟表 被 mysql 读入的数据一致。
举例如下:
```sql
explain select u.username, (select uid from iknow_team_user limit 1) uu from (select uid,username from iknow_team_user where uid=123) u;
+----+-------------+-----------------+--------+---------------+-----------+---------+------+--------+-------------+
| id | select_type | table           | type   | possible_keys | key       | key_len | ref  | rows   | Extra       |
+----+-------------+-----------------+--------+---------------+-----------+---------+------+--------+-------------+
|  1 | PRIMARY     | derived3        | system | NULL          | NULL      | NULL    | NULL |      1 |             | 
|  3 | DERIVED     | iknow_team_user | ref    | ituid_tid     | ituid_tid | 4       |      |      1 |             | 
|  2 | SUBQUERY    | iknow_team_user | index  | NULL          | ituid_tid | 8       | NULL | 414815 | Using index | 
+----+-------------+-----------------+--------+---------------+-----------+---------+------+--------+-------------+
3 rows in set (0.01 sec)
```
以下几点需要注意:
1. 如果是子查询 SUBQUERY,id 越大优先级越高,越先被执行,如下:
```sql
explain select * from iknow_team_user where iknow_team_user.uid=(select uid from iknow_team_user as tb where tb.uid=(select uid from iknow_team_user as tc where tc.uid=1));
+----+-------------+-----------------+------+---------------+-----------+---------+-------+------+--------------------------+
| id | select_type | table           | type | possible_keys | key       | key_len | ref   | rows | Extra                    |
+----+-------------+-----------------+------+---------------+-----------+---------+-------+------+--------------------------+
|  1 | PRIMARY     | iknow_team_user | ref  | ituid_tid     | ituid_tid | 4       | const |    1 | Using where              |
|  2 | SUBQUERY    | tb              | ref  | ituid_tid     | ituid_tid | 4       |       |    1 | Using where; Using index |
|  3 | SUBQUERY    | tc              | ref  | ituid_tid     | ituid_tid | 4       |       |    1 | Using index              |
+----+-------------+-----------------+------+---------------+-----------+---------+-------+------+--------------------------+
3 rows in set (0.00 sec)
```
2. id 如果相同,可以认为是一组,从上而下执行;所有组中,id 值越大,优先级越高,越先 被执行,如下:
3. union 操作 id 为 null
union 输出结果集会多出一行,union 的结果集总是放在一个临时表中,如&lt;union1,2&gt;,再从这个临时表中获取结果。此行 id 为 null。
如下:
```sql
explain select * from iknow_team_user where uid = 1 union select * from  iknow_team_user where uid=1213;
+----+--------------+-----------------+------+---------------+-----------+---------+-------+------+-------+
| id | select_type  | table           | type | possible_keys | key       | key_len | ref   | rows | Extra |
+----+--------------+-----------------+------+---------------+-----------+---------+-------+------+-------+
|  1 | PRIMARY      | iknow_team_user | ref  | ituid_tid     | ituid_tid | 4       | const |    1 |       |
|  2 | UNION        | iknow_team_user | ref  | ituid_tid     | ituid_tid | 4       | const |    1 |       |
| NULL | UNION RESULT | union1,2      | ALL  | NULL          | NULL      | NULL    | NULL  | NULL |       |
+----+--------------+-----------------+------+---------------+-----------+---------+-------+------+-------+
3 rows in set (0.00 sec)
```

### 2. select type 列

| val    | explain |
|--------|--------|
|SIMPLE  |简单 SELECT(不使用 UNION 或子查询)     |
|PRIMARY |查询中包含任何复杂查询,最外层的的SELECT  |
|SUBQUERY|在 select 或 where 中包含的子查询      |
|DERIVED |FROM 子句的子查询;若 union 包含在 from 子句的子查询中,外层的 select被标记为 DERIVED|
|UNION   | 在 select 或 where 中包含的子查询|
|UNION RESULT    | UNION结果|
|DEPENDENT UNION | UNION 中的第二个或后面的 SELECT 语句,取决于外面的查询 DEPENDENT SUBQUERY|
|DEPENDENT SUBQUERY | 子查询中的第一个 SELECT,取决于外面的查询|

举例如下:
### 3. type 列
即访问类型,可以理解为 mysql 在表里找出所需要行的方式 以下的访问类型,性能从最差到最好:
#### 3. 1. All
全表扫描,mysql 必须扫描从头到尾扫描整张表,找到所需要的行
例外:查询条件中使用了 limit，extra 列中显示使用了 distinct 或 not exists 等限定词
```sql
explain select * from iknow_team_user where tagList!='';
+----+-------------+-----------------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table           | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-----------------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | iknow_team_user | ALL  | NULL          | NULL | NULL    | NULL |    1 | Using where | 
+----+-------------+-----------------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.01 sec)
```
#### 3. 2. index
和全表扫描一样也是做遍历,不同的是它是做全索引遍历,按照索引的次序进行遍历
优点:避免排序
如果在 extra 中看到 using index 说明使用了覆盖索引
举例如下:
```sql
explain select id from iknow_team_user limit 10;
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+-------------+
| id | select_type | table           | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+-------------+
|  1 | SIMPLE      | iknow_team_user | index | NULL          | PRIMARY | 4       | NULL |    1 | Using index |
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+-------------+
1 row in set (0.00 sec)
```
#### 3. 3. range
有限制的索引扫描,比全索引扫描好一些,不用遍历全部索引,常见的:> < in or
举例如下:
```sql
explain select * from iknow_team_user where id < 5 limit 10;
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+------------------------------+
| id | select_type | table           | type  | possible_keys | key     | key_len | ref  | rows | Extra                        |
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+------------------------------+
|  1 | SIMPLE      | iknow_team_user | range | PRIMARY       | PRIMARY | 4       | NULL |    1 | Using where; Using temporary |
+----+-------------+-----------------+-------+---------------+---------+---------+------+------+------------------------------+
1 row in set (0.00 sec)
```
#### 3. 4. ref
非唯一性索引扫描,访问非唯一索引,或唯一索引的前缀时返回匹配这个单独值的所有行
```sql
explain select * from iknow_team_user where uid=1 limit 10;
+----+-------------+-----------------+------+---------------+-----------+---------+-------+------+-----------------+
| id | select_type | table           | type | possible_keys | key       | key_len | ref   | rows | Extra           |
+----+-------------+-----------------+------+---------------+-----------+---------+-------+------+-----------------+
|  1 | SIMPLE      | iknow_team_user | ref  | ituid_tid     | ituid_tid | 4       | const |    1 | Using temporary |
+----+-------------+-----------------+------+---------------+-----------+---------+-------+------+-----------------+
1 row in set (0.00 sec)
```
#### 3. 5. eq_ref
唯一性索引扫描,主键或唯一索引扫描中常见
#### 3. 6. const,system
当 mysql 对查询的某部分进行了优化,并转话为一个常量时使用这种访问类型 system 是 const 类型的特例,当查询的表中只有一行时为 system
举例如下:
```sql
explain select * from iknow_team_user where teamId=1 and uid=1 limit 10;
+----+-------------+-----------------+-------+-----------------------+-----------+---------+-------------+------+-------+
| id | select_type | table           | type  | possible_keys         | key       | key_len | ref         | rows | Extra |
+----+-------------+-----------------+-------+-----------------------+-----------+---------+-------------+------+-------+
|  1 | SIMPLE      | iknow_team_user | const | ituid_tid,ittid_utype | ituid_tid | 8       | const,const |    1 |       |
+----+-------------+-----------------+-------+-----------------------+-----------+---------+-------------+------+-------+
1 row in set (0.00 sec)
```
#### 3. 7. NULL
mysql 能够在查询过程中分解查询语句,甚至在执行环节无需再访问表或者索引 举例如下:
```sql
explain select min(id) from iknow_team_user limit 10;
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra                        |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------+
|  1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | Select tables optimized away |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------+
1 row in set (0.00 sec)
```
神奇!! !table、type、key 列都为 NULL
### 4. possible_keys 列 与 key 列
possible_keys 列表明该查询可能使用哪些索引,但一个查询仅能使用到一个索引,key 列显示了 mysql 使用了哪一个索引来优化对该表的访问。但若查询使用了覆盖索引,则仅出现在
key列中,举例如下:
### 5. key_len 列
显示了 mysql 在索引中使用的字节数,key_len是根据表定义算出来的,和具体数据无关。
如上图举例,uid、doc_id各四个字节,故 key_len为8
### 6. ref 列
表示表的连接匹配条件,即哪些列或者常量被用于查找索引列上的值,举例如下:
### 7. rows 列
表示 mysql 根据表统计信息及索引使用情况,估算的要找出结果需要读取的记录行数
### 8. Extra 列
Extra 列有多种取值,介绍如下四种
- using index
表明在 select 操作中使用了覆盖索引
- using where
表示 mysql 存储引擎收到结果后再进行过滤,举例如下:
- using temporary
表示 mysql 需要使用临时表来存储结果集,排序、分组常见,如下:
- using filesort
mysql 中无法利用索引完成的排序成为 filesort,如下
```sql
explain select teamId from iknow_team_user group by tagList limit 10;
+----+-------------+-----------------+------+---------------+------+---------+------+------+---------------------------------+
| id | select_type | table           | type | possible_keys | key  | key_len | ref  | rows | Extra                           |
+----+-------------+-----------------+------+---------------+------+---------+------+------+---------------------------------+
|  1 | SIMPLE      | iknow_team_user | ALL  | NULL          | NULL | NULL    | NULL |    1 | Using temporary; Using filesort |
+----+-------------+-----------------+------+---------------+------+---------+------+------+---------------------------------+
1 row in set (0.00 sec)
```
