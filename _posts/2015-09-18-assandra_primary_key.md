---
title: Cassandra primary key, clustering key and Secondary Index
description: >
  Cassandra primary key, clustering key and Secondary Index
layout: post
categories: Cassandra
---

#Cassandra primary key, clustering key和 Secondary Index


最近在Cassandra的使用过程中, 发现Cassandra的查询操作异常缓慢(花费了700~900ms), 经过排查后发现是使用了Secondary Index的原因.本文整理了primary key 和 Secondary Index在Cassandra中的存储方式, 也解释了为什么使用Secondary Index查询会非常缓慢.

#Primary Key
Cassandra的Primary Key由Partition Key和Clustering Key两部分组成. 先看只有Partition Key的情况.
例如, 如下建表语句:

```
CREATE TABLE sample (
  id text,
  name text,
  PRIMARY KEY (id)
);

```

该表的primary key就是其partition key. 在Cassandra中, 每一个partition key对应着一行, 所有的column都存储在该行中.如下图所示:

			+-------------+----------+-----------+------------+
			|  partition  |  column  |  column   |   column   |
			|    key 1    |    1     |     2     |      3     |
			+-------------+----------+-----------+------------+
			|  partition  |  column  |  column   |   column   |
			|    key 2    |    1     |     2     |      3     |
			+-------------+----------+-----------+------------+
例如, 在sample中插入若干记录, 查询该表:

```
SELECT * FROM sample;
```

得到如下结果:

```
 id  | name
-----+-------
 id3 | name3
 id1 | name1
 id2 | name2
```

然后, 再用Cassandra-Cli去查询, 得到如下结果:

```
	RowKey: id3
	=> (name=, value=, timestamp=1442491586006000)
	=> (name=name, value=6e616d6533, timestamp=1442491586006000)
	-------------------
	RowKey: id1
	=> (name=, value=, timestamp=1442491594600000)
	=> (name=name, value=6e616d6531, timestamp=1442491594600000)
	-------------------
	RowKey: id2
	=> (name=, value=, timestamp=1442491578108000)
	=> (name=name, value=6e616d6532, timestamp=1442491578108000)

	3 Rows Returned.
	
```

可以看到, 每一个partition key对应着一个RowKey, 所有的column都存放在这一行上.

#Clustering Key
Cassandra中所有的数据都只能根据Primary Key中的字段来排序, 因此, 如果想根据某个column来排序, 必须将改column加到Primary key中, 如 primary key (id, c1, c2 ,c3), 其中id时partition key, c1, c2 ,c3是Clustering Key.(如果想用id和c1作为partition key, 只需添加括号: primary key ((id, c1), c2 ,c3)). 

创建表:

```
CREATE TABLE sample3 (
  id text,
  gmt_create bigint,
  name text,
  score text,
  PRIMARY KEY ((id), gmt_create)
);
```
在该表中插入若干纪录后, 使用select *查询该表, 得到如下纪录:

```
id  | gmt_create | name  | score
-----+------------+-------+--------
 id3 |       1925 | name3 | score3
 id3 |       1926 | name4 | score4
 id1 |       1923 | name1 | score1
 id2 |       1924 | name2 | score2
```

可以看到cql表中有四条记录, 其中id=id3有两条记录, 那么是否意味着cassandra中有四行来保存这些记录呢?用Cssandra-Cli来查询:

```
	RowKey: id3
	=> (name=1925:, value=, timestamp=1442494070376000)
	=> (name=1925:name, value=6e616d6533, timestamp=1442494070376000)
	=> (name=1925:score, value=73636f726533, timestamp=1442494070376000)
	=> (name=1926:, value=, timestamp=1442494335821000)
	=> (name=1926:name, value=6e616d6534, timestamp=1442494335821000)
	=> (name=1926:score, value=73636f726534, timestamp=1442494335821000)
	-------------------
	RowKey: id1
	=> (name=1923:, value=, timestamp=1442494039343000)
	=> (name=1923:name, value=6e616d6531, timestamp=1442494039343000)
	=> (name=1923:score, value=73636f726531, timestamp=1442494039343000)
	-------------------
	RowKey: id2
	=> (name=1924:, value=, timestamp=1442494054817000)
	=> (name=1924:name, value=6e616d6532, timestamp=1442494054817000)
	=> (name=1924:score, value=73636f726532, timestamp=1442494054817000)

	3 Rows Returned.
	
```

可以看到, cassandra中只用3行来保存数据, 其中id=id3到两条记录被保存在同一行上. 事实上, 当存在clustering key时, 当partition key相同而clustering key不同时, 其纪录是保存在同一行上的, 而每一个column的name责由clustering的值 + ":" + 原来的column name构成(如sample3中, column score的name = 1923:score), sample3在cassandra中的存储结构如下图所示:

            +-------------+----------------+-------------------+----------------------+
            |             |  name = 1925:  |  name=1925:name   |    name=1925:score   |
            |             |  value =       |  value=6e616d6533 |  value=73636f726533  |
            |             |  timestamp     |     timestamp     |       timestamp      |
            |    id 3     +----------------+-------------------+----------------------+
            |             |  name = 1926:  |  name=1926:name   |    name=1926:score   |
            |             |  value=        |  value=6e616d6534 |  value=73636f726534  |
            |             |  timestamp     |     timestamp     |       timestamp      |
            +-------------+----------------+-------------------+----------------------+
            |             |  name = 1923:  |  name=1923:name   |    name=1923:score   |
            |    id 1     |  value =       |  value=6e616d6531 |  value=73636f726531  |
            |             |  timestamp     |     timestamp     |       timestamp      |
            +-------------+----------------+-------------------+----------------------+
            |             |  name = 1924:  |  name=1924:name   |    name=1924:score   |
            |    id 2     |  value =       |  value=6e616d6532 |  value=73636f726532  |
            |             |  timestamp     |     timestamp     |       timestamp      |
            +-------------+----------------+-------------------+----------------------+
            
#Secondary Index

从cassandra 0.7开始支持Secondary Index. 例如, 在sample3上创建Secondary Index:

```
create index idx_score on sample3(score);
```

而Secondary Index的原理类似于创建一张新的表, 该表的primary key就是索引的列.因此, 若原sample表中有数据:

```
{
    "id1": {
        "name" : "jim"
        "score": "60"
    },
    "id2": {
        "name" : "kate"
        "score": "60"
    }
}
```

索引表的内部结构类似于如下：

```
{
    "60": {
        "id1": data
        "id2": data
    }
}
```

与原表的不同之处在于: 原表的数据按照partition key分发到不同的节点进行存储, 当cassandra进行读取时, 任意一个节点返回数据即可(取决于读取时的一致性要求), 而secondary index 表则会存储自己节点上的数据, 因此, 按照secondary index进行读取时, 需要等待每一个节点读取完毕, 这个过程相比于primary key读取的过程会比较缓慢.(当并发非常高时, 就会造成大量的random I/O, 导致查询非常缓慢.)

因此, 当secondary index的column是类似于'国家'这样有许多重复纪录的字段时, 读取时相当高效的(因为n个节点都进行一个磁盘搜素, 时间复杂度O(n), 然而查询到的纪录数有可能是x条, 大多数情况下x>n, 平均下来时非常高效的), 但是当secondary index的column是类似于email这样几乎每个人唯一的字段时, 是非常低效的, 因为按照email进行查询是, 每一次查询几乎只能命中一条纪录, 而secondary index搜索的时间复杂度则是O(n)。
