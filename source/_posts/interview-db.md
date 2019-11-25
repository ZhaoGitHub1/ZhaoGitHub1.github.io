---
title: 面试之---数据库相关
categories: interview
tags: 
  - 总结
  - 数据库
date: 2018-09-16 21:03:51
description: 包含关系型数据库mysql、oracle等相关面试知识汇总
---

** {{ title }}：** <Excerpt in index | 首页摘要>

<!-- more -->
<The rest of contents | 余下全文>

![](interview-db/interview.png)

> 持续维护java面试题系列博文，发现好的面试题会更新进来。总结的并不只是如何回答面试官，而是这道题涉及到的知识点，明白了相关知识点和面试官要问的点，回答起来就不是问题了。

## 综合

### 解释下什么是脏读、不可重复读和幻读？

## Mysql

### Mysql有哪几种事务隔离级别？

- 读未提交（read uncommitted）
  - 存在脏读、不可重复读和幻读
- 读已提交（read committed）
  - 存在不可重复读和幻读
- 可重复读（repeatable read）
  - 存在幻读
- 序列化（serializable）

### Mysql的默认事务隔离级别是什么？

可重复读（repeatable read）

### 为什么不推荐使用`select *`

> 如果查询列可通过索引节点中的关键字直接返回，则该索引称之为覆盖索引。
>
> 覆盖索引可以减少数据库IO，将随机IO变为顺序IO，提高查询新能。

避免使用`select *`语句，使覆盖索引的概率加大，以及减少数据传输。 

例如：`select name,age from t_user where name  = ?`这个查询命中了`idx_name_age`的组合索引，则查找数据时不需要遍历到叶子结点拿到完整数据就可以提前返回，减少了数据库的IO次数。

### select * from t_user where name like "张三%"，一定会用到name的索引吗？

不一定，当name的索引离散性很差时，优化器会放弃索引，进行全表扫描。



## Oracle