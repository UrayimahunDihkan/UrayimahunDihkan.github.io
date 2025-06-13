---
title: 到底数据量达到多少时要考虑分库分表?
date: 2024-08-02 22:04:36
tags:
---

经常有人会说500w或两千万就应该考虑分表，但是实际工作中做决策时需要负责任的，需明察其理。

其实, 如果了解[B+树](https://zhida.zhihu.com/search?content_id=238285156&content_type=Article&match_order=1&q=B%2B树&zhida_source=entity)数据结构，我们自己就可以计算和预判数据量达到多少后数据库性能可能会下降，并且按业务现有的数据评估要不要分表。

首先复习一下B+树。一个[主键索引](https://zhida.zhihu.com/search?content_id=238285156&content_type=Article&match_order=1&q=主键索引&zhida_source=entity)的B+树数据结构大概长这样：

![img](https://pic4.zhimg.com/v2-60dbb180e987238fc31afb9f14078711_1440w.jpg)

如所见，这些大框(不是有数字的小框)都我们称之为节点或“页”，每页大小在[mysql](https://zhida.zhihu.com/search?content_id=238285156&content_type=Article&match_order=1&q=mysql&zhida_source=entity)默认为16kb(官方说这是mysql做了大量的实验和调优后得出来的合适值)。请看第一页，假设索引的类型用的是bingint类型，那么"50"这一格大小为8byte，"50"隔壁的格存储的是"50"到"70"之间的数据页的地址，mysql给这个地址大小为6byte。
所以一个页可以存储:
16kb/(8byte+6byte)=1170(个bigint)
假设一行记录占1kb，在三层b+[聚簇索引](https://zhida.zhihu.com/search?content_id=238285156&content_type=Article&match_order=1&q=聚簇索引&zhida_source=entity)树中我们可以存1170 × 1170 × 16kb/1kb = 21902400(条记录)

很多建议两千多万分表这个说法来自于这里。

然而，不是每一张业务表的记录大小都等于1kb，1kb空间足以让线上大多数业务的一条数据撑死。
比如有这么一条记录，字段及类型长这样：

| 字段     | id     | name     | age   | gender  |
| -------- | ------ | -------- | ----- | ------- |
| 类型     | bigint | char(20) | int   | tinyint |
| 占用空间 | 8byte  | 21byte   | 4byte | 1byte   |

这条记录的大小粗略的等于34个字节，如果还是拿bigint类型的id索引算，那么这个三层B+树能存1170 × 1170 × 16kb/34byte = 659809800(条记录)

就这个表而言，可以认为数据增长到6.59亿条之前都不太会有性能上的问题。

因此在业务中面对那么多天马星空的表结构、索引的情况、单条数据平均大小，并不应该死认500w或者2000w，因为阿里开发手册上的建议是来自于阿里自己的数据，跟我们的未必一样。
