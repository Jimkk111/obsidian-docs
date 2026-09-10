---
title: MySql
tags:
  - MOC
---

# MySql

上级：[[web后端开发/数据库/数据库|数据库]]

结构参考 [MySQL 8.0 官方参考手册](https://dev.mysql.com/doc/refman/8.0/en/)。

## 基础

- [[web后端开发/数据库/MySql/介绍|介绍]]：数据库与 SQL 概念、MySQL 体系结构、SQL 语言分类
- 数据定义：[[web后端开发/数据库/MySql/DDL|DDL]]、[[web后端开发/数据库/MySql/数据类型|数据类型]]
- 数据操纵：[[web后端开发/数据库/MySql/DML|DML]]
- 数据查询：[[web后端开发/数据库/MySql/DQL|DQL]]、[[web后端开发/数据库/MySql/多表查询|多表查询]]、[[web后端开发/数据库/MySql/函数|函数]]
- 权限与事务：[[web后端开发/数据库/MySql/DCL|DCL]]、[[web后端开发/数据库/MySql/事务|事务]]

## 进阶

- [[web后端开发/数据库/MySql/存储引擎|存储引擎]]：InnoDB 架构与逻辑存储结构、InnoDB vs MyISAM
- [[web后端开发/数据库/MySql/索引|索引]]：B+ 树、聚簇/二级索引、最左前缀、索引失效、EXPLAIN
- [[web后端开发/数据库/MySql/MVCC|MVCC]]：隐藏字段、undo log 版本链、ReadView、RC/RR 差异
- [[web后端开发/数据库/MySql/锁|锁]]：全局/表级/行级锁、间隙锁与临键锁、死锁、乐观锁与悲观锁
- [[web后端开发/数据库/MySql/存储过程|存储过程]]：变量、条件循环、游标、handler
