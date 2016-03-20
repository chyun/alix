---
title: Cassandra write path(2)
description: >
  Cassandra write path
layout: post
categories: Cassandra
---

#SSTable

SSTable是磁盘中对应CQL的存储结构, 如前所述, 当满足一定条件时, Cassandra会将Memtable中的数据刷到SSTable中.

```
SSTable是不可修改的
SSTable是按照token排序的
同时包含了Memtable中的数据被刷到SSTable时Memtable当时的状态信息
```

#Compaction
如前所述, 当update, delete等操作到来时, Cassandra并没有执行真正的update/delete, 而只是将他们像insert一样, append到Memtable中. 而这些数据又会被刷到SSTable中, 因此Cassandra中包含了许多没有用的数据, Cassandra需要定期对这些数据进行处理. 处理方式就是Cassandra会定期将众多的SSTable合并成一个SSTable, 这个过程叫做Compaction. 

进行Compaction的目的主要是以下两个:

```
减少SSTable的数目, 以增加读取时的速度
    由于Cassandra写入的方式是append, Cassandra在读取数据时, 需要读取所有SSTable中的数据, 并加以整合
释放空间

```

Compaction的过程如下图所示:

![enter image description here][1]

在Compaction的过程中, Cassandra主要完成一下几个操作:

```
合并key
   将primary key相同的行, 合并成一行
整合column
   整理column中的数据, 只保留timestamp最新的数据
丢弃tombstone(Tombstone是Memtable中数据被删除的标记)
   当tombstone的存在时间已经超过gc_grace_seconds(默认值是10天, 主要是为了防止已经删除的数据复活, 参考https://wiki.apache.org/cassandra/DistributedDeletes)
```


[1]: https://github.com/chyun/Blog/blob/gh-pages/images/2015-09-11-compaction.png?raw=true
