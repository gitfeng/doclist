---
title: MySQL_查询优化-2/2
categories: 技术
tags: [mysql]
description:  mysql使用过程中的一些优化注意事项,如'大数据翻页'，'随机结果获取'等

---

## 随机结果获取

ref: [参考链接](http://imysql.com/2014/07/04/mysql-optimization-case-rand-optimize.shtml)

```sql
SELECT id FROM table ORDER BY RAND() LIMIT n;
```

改造成下面这个：

```sql
SELECT id FROM table t1 JOIN (SELECT RAND() * (SELECT MAX(id) FROM table) AS nid) t2 ON t1.id >
t2.nid LIMIT n;
```

如果想要达到完全随机，还可以改成下面这种写法：

```sql
SELECT id FROM table t1 JOIN (SELECT round(RAND() * (SELECT MAX(id) FROM table)) AS nid FROM table
        LIMIT n) t2 ON t1.id = t2.nid;
```

## 大数据翻页

### 不带 where 的优化

思路：

- 消去limit $pn, $rn 的第一个参数， 即变成 limit $rn;
- 条件转成主键索引

```sql
select * from t where id > 99 order by id asc limit $page_size ;
```

### 带 where 的优化
ref: [参考链接](http://imysql.com/2014/07/26/mysql-optimization-case-paging-optimize.shtml)

```sql
SELECT * FROM `t1` ORDER BY id DESC LIMIT 935500, 100
```

子查询优化：
```sql
-- 采用子查询的方式优化，在子查询里先从索引获取到最大id，然后倒序排，再取10行结果集
-- 注意这里采用了2次倒序排，因此在取LIMIT的start值时，比原来的值加了10，即935510，否则结果将和原来的不一致

SELECT * FROM (SELECT * FROM `t1` WHERE id > ( SELECT id FROM `t1` ORDER BY id DESC LIMIT 935510, 1)
        LIMIT 10) t ORDER BY id DESC;
```

left-join 优化：
```sql
-- 采用INNER JOIN优化，JOIN子句里也优先从索引获取ID列表，然后直接关联查询获得最终结果，这里不需要加10
SELECT * FROM (SELECT * FROM `t1` WHERE id > ( SELECT id FROM `t1` ORDER BY id DESC LIMIT 935510, 1)
        LIMIT 10) t ORDER BY id DESC;
```

两种优化方式效果统计：

|    |大分页，带WHERE| 大分页，不带WHERE | 小分页，带WHERE | 小分页，不带WHERE  |
|----|--------------|-----|------|--------|
| 子查询优化    | 28.20% | 10.60%| 24.90% | 554.40% |
| INNER JOIN优化| 30.80% | 30.20% | 156.50%| 11.70% |

