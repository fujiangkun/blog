---
title: "MySQL 查询优化"
date: 2020-04-19T17:53:48+08:00
lastmod: 2020-04-19T18:01:43+08:00
draft: true
categories:
    - 数据库 
tags:
    - mysql
---

在日常开发过程中，免不了会遇到程序中出现查询慢的情况，那我们需要怎样做才能让我们的查询更快一步呢？这就需要学习 MySQL 查询优化相关的知识了。

<!--more-->

#### MySQL 查询流程

1. 客户端发送一条查询给服务器
2. 服务器先检查缓存、命中则返回，否则下一步
3. 服务器进行 SQL 解析、预处理、由优化器生成执行计划
4. 依据执行计划，调用存储引擎 API 执行查询
5. 将结果返回给客户端

#### Explain 分析

[MySQL EXPLAIN ](./mysql_explain.md)

#### 查询优化

1、不是全表查询，只查询需要的字段，尽可能避免大量的 `select *` 操作，但是有时候 `select *` 能够利用 MySQL 的缓存机制对数据进行缓存，所有也需要相应的权衡。

2、查看是否扫描了不需要的行：通过 `explain` 进行查看查询扫描的类型（`type` 字段)：从全表扫描、索引扫描、范围扫描、唯一索引查询、常数引用，速度由慢到快，扫描行数（`rows` 字段）也由多到少。尽可能地减少获取数据所需要扫描的行数，可以，能有效提升查询速度。

3、如果一个查询能解决的问题，尽量不要拆成多个查询。

4、查询切分：将一次需要查询大批量数据的切分成多次查询，方式查询数据过多导致数据加锁操作。

5、分解关联查询：将关联查询分解为多个单查询；此处主要利用的是 MySQL 的缓存、同时减少锁竞争；

6、关联查询小表驱动大表：先通过条件查询小表数据，再去关联大表进行数据查询，以减少扫描的数据量。如果执行计划没有按照指定的关联顺序执行，可以通过 `STRAIGHT_JOIN` 关键字强制按照我们 SQL 的关联顺序执行。

7、LIMIT 分页优化：当 limit 的 offset 达到一定程度的时候，扫描的数据量就会很多，这时候会影响分页查询，可以通过获取上一次查询的最大 ID 作为筛选条件，减少扫描的数据行。

```mysql
select * from tb where cond limit 10000,20;
select * from tb where cond and id > 100000 limit 20; -- 性能更优
```

