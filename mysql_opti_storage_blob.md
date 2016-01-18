---
title: MySQL_存储优化-InnoDB表BLOB列
categories: 技术
tags: [mysql]
description: 优化InnoDB表BLOB列的存储效率

---


ref : [优化InnoDB表BLOB列的存储效率](http://imysql.com/2014/09/28/mysql-optimization-case-blob-stored-in-innodb-optimization.shtml)

### InnoDB引擎存储格式的几个要点：

1. InnoDB可以选择使用共享表空间或者是独立表空间方式，建议使用独立表空间，便于管理、维护。启用 innodb_file_per_table 选项，5.5以后可以在线动态修改生效，并且执行 ALTER TABLE xx ENGINE = InnoDB 将现有表转成独立表空间，早于5.5的版本，修改完这个选项后，需要重启才能生效；
2. InnoDB的data page默认16KB，5.6版本以后，新增选项 innodb_page_size 可以修改，在5.6以前的版本，只能修改源码重新编译，但并不推荐修改这个配置，除非你非常清楚它有什么优缺点；
3. InnoDB的data page在有新数据写入时，会预留1/16的空间，预留出来的空间可用于后续的新纪录写入，减少频繁的新增data page的开销；
4. 每个data page，至少需要存储2行记录。因此理论上行记录最大长度为8KB，但事实上应该更小，因为还有一些InnoDB内部数据结构要存储；
5. 受限于InnoDB存储方式，如果数据是顺序写入的话，最理想的情况下，data page的填充率是15/16，但一般没办法保证完全的顺序写入，因此，data page的填充率一般是1/2到15/16。因此每个InnoDB表都最好要有一个自增列作为主键，使得新纪录写入尽可能是顺序的；
6. 当data page填充率不足1/2时，InnoDB会进行收缩，释放空闲空间；
7. MySQL 5.6版本的InnoDB引擎当前支持COMPACT. REDUNDANT、DYNAMIC、COMPRESSED四种格式，默认是COMPACT格式，COMPRESSED用的很少且不推荐（见下一条），如果需要用到压缩特性的话，可以直接考虑TokuDB引擎；
8. COMPACT行格式相比REDUNDANT，大概能节省20%的存储空间，COMPRESSED相比COMPACT大概能节省50%的存储空间，但会导致TPS下降了90%。因此强烈不推荐使用COMPRESSED行格式；
9. 当行格式为DYNAMIC或COMPRESSED时，TEXT/BLOB之类的长列（long column，也有可能是其他较长的列，不一定只有TEXT/BLOB类型，看具体情况）会完全存储在一个独立的data page里，聚集索引页中只使用20字节的指针指向新的page，这就是所谓的off-page，类似ORACLE的行迁移，磁盘空间浪费较严重，且I/O性能也较差。因此，强烈不建议使用BLOB. TEXT、超过255长度的VARCHAR列类型；
10. 当InnoDB的文件格式（innodb_file_format）设置为Antelope，并且行格式为COMPACT 或 REDUNDANT 时，BLOB. TEXT或者长VARCHAR列只会将其前768字节存储在聚集索页中（最大768字节的作用是便于创建前缀索引/prefix index），其余更多的内容存储在额外的page里，哪怕只是多了一个字节。因此，所有列长度越短越好；
11. 在off-page中存储的BLOB、TEXT或者长VARCHAR列的page是独享的，不能共享。因此强烈不建议在一个表中使用多个长列。

### 建议
综上，如果在实际业务中，确实需要在InnoDB表中存储BLOB、TEXT、长VARCHAR列时，有下面几点建议：

1. 尽可能将所有数据序列化、压缩之后，存储在同一个列里，避免发生多次off-page；
2. 实际最大存储长度低于255的列，转成VARCHAR或者CHAR类型（如果是变长数据二者没区别，如果是定长数据，则使用CHAR类型）；
3. 如果无法将所有列整合到一个列，可以退而求其次，根据每个列最大长度进行排列组合后拆分成多个子表，尽量是的每个子表的总行长度小于8KB，减少发生off-page的频率；
4. 上述建议是在data page为默认的16KB前提下，如果修改成8KB或者其他大小，请自行根据上述理论进行测试，找到最合适的值；
5. 字符型列长度小于255时，无论采用CHAR还是VARCHAR来存储，或者把VARCHAR列长度定义为255，都不会导致实际表空间增大；
6. 一般在游戏领域会用到比较多的BLOB列类型，游戏界同行可以关注下。


