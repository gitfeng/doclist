---
title: 如何阅读 Redis 源码
date: 2016-01-28 15:01:00
categories: 技术
tags: [redis]
description:  较合理的 Redis 源码阅读顺序，希望可以给对 Redis 有兴趣并打算阅读 Redis 源码的朋友带来一点帮助。

---

ref: http://blog.huangz.me/diary/2014/how-to-read-redis-source-code.html

## 第1步：阅读数据结构实现
|adlist.c    |    |
|ae.c    |    |
|ae\_epoll.c    |    |
|ae\_evport.c    |    |
|ae\_kqueue.c    |    |
|ae\_select.c    |    |
|anet.c    |    |
|aof.c    |    |
|bio.c    |    |
|bitops.c    |    |
|blocked.c    |    |
|cluster.c    |    |
|config.c    |    |
|crc16.c    |    |
|crc64.c    |    |
|db.c    |    |
|debug.c    |    |
|dict.c    |    |
|endianconv.c    |    |
|hyperloglog.c    |    |
|intset.c    |    |
|lzf\_c.c    |    |
|lzf\_d.c    |    |
|memtest.c    |    |
|multi.c    |    |
|networking.c    |    |
|notify.c    |    |
|object.c    |    |
|pqsort.c    |    |
|pubsub.c    |    |
|rand.c    |    |
|rdb.c    |    |
|redis-benchmark.c    |    |
|redis-check-aof.c    |    |
|redis-check-dump.c    |    |
|redis-cli.c    |    |
|redis.c    |    |
|release.c    |    |
|replication.c    |    |
|rio.c    |    |
|scripting.c    |    |
|sds.c    |    |
|sentinel.c    |    |
|setproctitle.c    |    |
|sha1.c    |    |
|slowlog.c    |    |
|sort.c    |    |
|syncio.c    |    |
|t\_hash.c    |    |
|t\_list.c    |    |
|t\_set.c    |    |
|t\_string.c    |    |
|t\_zset.c    |    |
|util.c    |    |
|ziplist.c    |    |
|zipmap.c    |    |
|zmalloc.c    |    |
