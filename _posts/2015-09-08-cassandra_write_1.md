---
title: Cassandra write path(1)
description: >
  Cassandra write path.
layout: post
categories: cassandra
---

#Cassandra写数据过程

Cassandra写数据的速度非常快, 其原因就在于Cassandra是一个基于日志结构的存储引擎, Cassandra对数据的操作全部采用append的方式. 当Cassandra的任何一个节点, 接收到写请求时, 其写数据的整个过程如下图所示:

![enter image description here][1]

```
1. 将新记录写入CommitLog;
2. 将新纪录写入Memtable;
3. 在特定的时间, 将Memtable中的数据刷入到SSTables, 清空JVM Heap和CommitLog;
4. 在特定的时间, Cassandra Compaction将SSTables合并.
```

##CommitLog
如上图所示, 当Cassandra接收到写请求时, 需要将数据写入到Memtable中, 然而由于memtable是驻留在内存中的, 为了防止节点故障后的数据丢失, Cassandra在将数据写入到Memtable前, 会先将数据append到CommitLog中. 当节点从故障中恢复时, 会从CommitLog中读取数据.
关于CommitLog有两个比较重要的配置项(在cassandra.yaml中):

```
commitlog_segment_space_in_mb
	单个commitlog segment所能占用的最大空间
commitlog_total_space_in_mb
	commitlog所能占用的总空间, 当commitlog占用的空间达到该值, 最旧的commitlog segment所对应的Memtable中的数据将被刷到SSTable中
```
当Memtable中的数据将被刷到SSTable后, 对应的commitlog segment会被标记为flushed, 并在稍后回收重用.

此外, 为了加快写CommitLog的速度, Cassandra写入commitlog的操作也不是每次写请求就写一次磁盘的, Cassandra提供了两种写入策略, periodic或者batch, 可配置commitlog_sync参数来实现.

```
periodic
	这是Cassandra的默认配置, 当写请求出现时，Cassandra会先将需要写入commitlog的数据先写在内存中, 每隔一段时间(commitlog_sync_period_in_ms, 默认是10000), Cassandra会将一个同步请求加入到一个队列中, Cassandra会立即执行该同步操作, 将数据同步到磁盘中(也就是commitlog).
batch
	Cassandra会定时(commitlog_sync_batch_window_in_ms， 默认值50)将内存中的数据批量写入到磁盘中(也就是commitlog), 在每一次批量写操作发出后, 并在该任务完成前, 向客户端同步本次批量写操作.
	
```

关于CommitLog有一个最佳实践, 给Cassandra的commitlog和数据分配不同的磁盘(在cassandra.yaml中配置, 默认情况下commitlog和data都在目录/var/lib/cassandra下), 减少磁头的寻道时间.

##memtable
memtable是一张驻留在内存中的cql表:

```
-每一个节点上, 每一个keyspace下的cql表都对应着一个memtable
-memtable提供了较为快速的读写操作
-memtable所有写数据都是append的形式
-memtable中的数据是按照token排序的, 关于这一点可用token函数来验证, 如下图所示.
```

![enter image description here][2]

Cssandra会在特定的时间, 将memtable中的数据刷到SSTable中, 当满足下文中的任意条件时, 就会触发该操作:

```
1. commitlog_total_space_in_mb(这一点已在前文提到过)
2. memtable_total_space_in_mb
	当memtable占用的总空间超过memtable_total_space_in_mb(默认是jvm空间的1/3)时, 把最旧的commitlog segment所对应的Memtable中的数据刷到SSTable中
3. nodetool flush命令
```

关于Cssandra将memtable中的数据刷到SSTables,和SSTables的合并, 在一下篇继续介绍.

[1]: https://github.com/chyun/Blog/blob/gh-pages/images/2015-09-08-cassandra-write.png?raw=true
[2]: https://github.com/chyun/Blog/blob/gh-pages/images/2015-09-10-memtable-sort.png?raw=true