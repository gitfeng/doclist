---
title: Redis 类功能 list
date: 2016-03-28 15:01:00
categories: 技术
tags: [redis]
description:  redis源码各类文件功能说明，用于在起始阶段建议概念
---

ref: http://www.cnblogs.com/liping13599168/archive/2011/04/12/2013094.html

## 开始
 redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)和zset(有序集合)。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

## 各类功能说明

| 文件/类| 功能 |
|-------|------|
|adlist.c    | 用于对list的定义，它是个双向链表结构   |
|ae.c    |  用于Redis的事件处理，包括句柄事件和超时事件。  |
|ae_epoll.c    |  附录1  |
|ae_evport.c    | 附录1   |
|ae_kqueue.c    | 附录1   |
|ae_select.c    | 附录1   |
|anet.c    | 作为Server/Client通信的基础封装，包括anetTcpServer,anetTcpConnect,anetTcpAccept,anetRead,anetWrite等等方法。   |
|aof.c    |  aof，全称为append only file，作用就是记录每次的写操作,在遇到断电等问题时可以用它来恢复数据库状态。但是他不是bin的,而是text的。一行一行,写得很规范.如果你是一台redis,那你也能人肉通过它恢复数据。  |
|asciilogo.h | |
|bio.c    |    |
|bitops.c    |    |
|blocked.c    |    |
|cluster.c    |    |
|config.c    | 用于将配置文件redis.conf文件中的配置读取出来的属性通过程序放到server对象中。在main函数（server服务主入口点处）可以发现里面调用loadServerConfig(char *filename)方法，这个方法就是使用config.c里面的方法实现。具体会在后面的系列中详细介绍。   |
|crc16.c    |    |
|crc64.c    |    |
|db.c    | 对于Redis内存数据库的相关操作。   |
|debug.c    | 用于调试使用。   |
|dict.c    |  主要对于内存中的hash进行管理  |
|endianconv.c    |    |
|fmacros.h |用于Mac下的兼容性处理。 |
|help.h | 辅助于命令的提示信息，作用于redis-cli.exe可执行文件中。 |
|hyperloglog.c    |    |
|intset.c    |  整数范围内的使用set，并包含相关set操作。  |
|lzf.h| 附录2 |
|lzfP.h | 附录2 |
|lzf_c.c    | 附录2    |
|lzf_d.c    | 附录2    |
|memtest.c    |    |
|mkreleasehdr.sh| |
|multi.c    |  用于事务处理操作。  |
|networking.c    | 网络协议传输方法定义相关的都放在这个文件里面了。包括让Client连接上Server，让Slave挂接到Master，已经Server/Client之间的信息交互的实现等等。   |
|notify.c    |    |
|object.c    | 用于创建和释放redisObject对象   |
|pqsort.c    | 关于排序算法，sort.c具体作为Redis场景下的排序实现。   |
|pubsub.c    | 用于订阅模式的实现，有点类似于Client广播发送的方式。   |
|rand.c    |    |
|rdb.c    |   对于Redis本地数据库的相关操作，默认文件是dump.rdb（通过配置文件获得），包括的操作包括保存，移除，查询等等。 |
|redis-benchmark.c    |  用于redis性能测试的实现。见附4  |
|redis-check-aof.c    |  用于更新日志检查的实现。  |
|redis-check-dump.c    |  用于本地数据库检查的实现。  |
|redis-cli.c    | 客户端程序的实现。   |
|redis.c    | 服务端程序的实现。   |
|redisassert.h | |
|release.c    | 用于发布使用。   |
|replication.c    | 用于主从数据库的复制操作的实现。   |
|rio.c    |    |
|scripting.c    |    |
|sds.c    |  用于对字符串的定义  |
|sentinel.c    |    |
|setproctitle.c    |    |
|sha1.c    | 有关于sha算法的实现。   |
|slowlog.c    |    |
|solarisfixes.h | Solaris系统的兼容性实现。 |
|sort.c    |  关于排序算法，sort.c具体作为Redis场景下的排序实现。  |
|syncio.c    | 用于同步Socket和文件I/O操作。   |
|t_hash.c    | 见附3   |
|t_list.c    | 见附3   |
|t_set.c    |  见附3  |
|t_string.c    | 见附3   |
|t_zset.c    |  见附3  |
|testhelp.h | 一个C风格的小型测试框架。 |
|util.c    |  关于通用工具的方法实现。  |
|version.h | Redis版本号定义。|
|ziplist.c    | ziplist是一个类似于list的存储对象。它的原理类似于zipmap。   |
|zipmap.c    | zipmap是一个类似于hash的存储对象。在新建一个hash对象时开始是用zipmap（又称为small hash）来存储的。这个zipmap其实并不是hash table但是zipmap相比正常的hash实现可以节省不少hash本身需要的一些元数据存储开销，如果field或者value的大小超出一定限制后，redis会在内部自动将zipmap替换成正常的hash实现。  |
|zmalloc.c    |  关于Redis的内存分配的封装实现。  |

## 附录
1. 在网络相关操作中，定义了一组公共操作接口：aeApiCreate，aeApiFree，aeApiAddEvent，aeApiDelEvent，aeApiPoll，aeApiName方法。在ae_epoll.c、ae_kqueue.c和ae_select.c中，分别实现了基于epoll/kqueue和select系统调用的接口。系统调用的选择顺序依次为epoll，kqueue，select。
2. 对于本地数据库的保存，使用的是LZF压缩算法，很神奇，算法只有200-300行的代码。
3. hash，list，set，string，zset在Server/Client中的应答操作。主要通过redisObject进行类型转换。
4. 用于redis性能测试的实现。请看main方法以下设置：

```
config.debug = 0;
config.numclients = 50;
config.requests = 10000;
config.liveclients = 0;
config.el = aeCreateEventLoop();
aeCreateTimeEvent(config.el,1,showThroughput,NULL,NULL);
config.keepalive = 1;
config.donerequests = 0;
config.datasize = 3;
config.randomkeys = 0;
config.randomkeys_keyspacelen = 0;
config.quiet = 0;
config.loop = 0;
config.idlemode = 0;
config.latency = NULL;
config.clients = listCreate();
config.hostip = "127.0.0.1";
config.hostport = 6379;
config.hostsocket = NULL;
 
parseOptions(argc,argv);
config.latency = zmalloc(sizeof(long long)*config.requests);

```
默认性能测试中的客户端数量为50个，并行发送的请求有10000条，你也可以通过redis-benchmark命令行参数进行设置。


