﻿# MySQL
数据库这们课程很早很早以前在本科阶段就讲了。当时这门课的广度非常广，从ER图，范式到事务，存储过程，关于数据库这一块想要完全精通还是十分的困难的。但有些很关键很重要的一些基础点还是必须要了解的，这对后面系统的性能和表结构设计等都是十分有用的。我就从面试常问和自己总结一些关键点记录下来吧。本篇笔记吧各个知识点并没有展开来讲。后续有时间会逐步补充。

## MySQL两种引擎和区别
mySQL包括两种引擎，MyISAM和InnoDB
他们区别包括：
**聚簇索引和非聚簇索引**
**支不支持事务（MVCC）**
**表锁和行级锁**
**支不支持外键**
这里面每种具体的细节，包括什么是聚簇非聚簇，行锁又分为共享锁和排他锁（in share model）(for update)等都是要自己去理解并整明白的。

## 索引
索引这又是必问的一个方向。包括
索引的数据结构（B+树）为什么不是B树。
B+树中的聚簇，非聚簇等等。
还有索引的种类，在innerDB中，单列索引，联合索引（最左匹配），辅助索引（辅助索引的原理）。
什么情况下要对一列单独建立索引。
索引的优劣等等。
这些都是要熟悉的。
## 事务
事务的四个特点（ACID）
事务的隔离级别。（未提交读，提交读，可重复读。串行化等等）
每种隔离级别避免了什么问题出现，并举例说明

## SQL优化
数据库的优化分很多的方面，包括索引，数据库本身优化
还有值得一提的是SQL语句本身的优化，常使用的包括用连接取代子查询，减少使用in等。
这些都需要举出例子的。可能面试官还会问你有没有用explain观察SQL语句的具体执行过程。

## 分库分表
包括纵向切分，水平切分等，分表考虑的因素等，怎么把用户分到合适的表之上。

## 存储过程，触发器，范式等
这些在实际中用的不是很多，但是也要知道个大概

这里面只是我根据几个大点列出来的需要学会的地方，具体针对每一点都会有很多很多的的知识点，参照这个提纲都要掌握才可以。
