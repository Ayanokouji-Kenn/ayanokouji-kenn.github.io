---
title: Unique Constraint Failded
date: 2016-07-20 10:33:27
tags: problems
---
-  今天碰到个奇怪的问题
数据库建表的时候，主键是这样的`_id INTEGER PRIMARY KEY AUTOINCREMENT`
插入第一条数据没问题，插入第2条数据的时候却弹了个Unique Constraint Failded xxx._id,
说主键的唯一性约束出错？？
通过debug发现插入的第一条数据的_id为0，我们知道设置的主键应该是从1开始自增的，那0是怎么回事？

- 解决问题
排查发现是**插入的时候多了一个表中没有的字段**，此时插入不会出错，只是会插在0的位置，如果继续插入不存在的字段，sqlite还会插在0的位置，就导致了唯一性约束出错。
将插入的代码改掉就好了