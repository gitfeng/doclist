---
title: MySQL_5.7部分新特性
date: 2016-01-17 14:21:00
categories: 技术
tags: [mysql]
description:  mysql 5.7中一些有意思的新特性

---


### MRR优化（Multi-Range Read）

MRR优化的目的是减少磁盘的随机访问。适用于range、ref和eq_ref类型。
优势：

- 使得数据访问变的较为顺序。在查询辅助索引时，先对查到的结果按照主键进行排序，并按照主键排序的顺序进行书签查找。
- 减少缓冲池中页被替换的次数。
- 批量处理对键值的查询操作。

联合索引(a,b)

```sql
select * from t where a>=1000 and a<2000 and b=1000;
```

如果用了MRR优化，那么优化器会先对查询条件进行拆分，然后再进行数据的查询。
拆分成（1000,1000），（1001,1000）… （1999,1000）

### ICP优化（Index Condition Pushdown）

有联合索引（group,last_name,first_name）

```sql
select * from t where group=’1234’ and lastname like ‘%ch%’ and first_name like ‘%zhao%’;
```

该sql只能通过联合索引的group来定位到所有符合条件的行所在的主键，必须取出所有group=’1234’的行记录然后过滤lastname like ‘%ch%’ and first_name like ‘%zhao%’

如果开启ICP优化，在存储引擎层就可以过滤lastname like ‘%ch%’ and first_name like ‘%zhao%’，然后使用过滤后的索引得到对应的主键，减少了大量无效的IO操作。
