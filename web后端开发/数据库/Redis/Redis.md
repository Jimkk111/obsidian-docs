---
title: Redis
tags:
  - MOC
---

# Redis

上级：[[web后端开发/数据库/数据库|数据库]]

结构参考 [Redis 官方文档](https://redis.io/docs/latest/)，命令速查见 [Commands](https://redis.io/docs/latest/commands/)。

## 基础

- [[web后端开发/数据库/Redis/介绍|介绍]]：NoSQL 概述、Redis 简介与特性、安装与客户端连接
- [[web后端开发/数据库/Redis/通用命令|通用命令]]：KEYS/SCAN、DEL、EXPIRE、TTL、TYPE，key 命名规范
- 五种基本类型：[[web后端开发/数据库/Redis/String|String]]、[[web后端开发/数据库/Redis/Hash|Hash]]、[[web后端开发/数据库/Redis/List|List]]、[[web后端开发/数据库/Redis/Set|Set]]、[[web后端开发/数据库/Redis/ZSet|ZSet]]
- [[web后端开发/数据库/Redis/其他类型|其他类型]]：BitMap、HyperLogLog、GEO、Stream

## 进阶

- [[web后端开发/数据库/Redis/Java客户端|Java客户端]]：Jedis / Lettuce / Spring Data Redis，RedisTemplate 与序列化
- [[web后端开发/数据库/Redis/持久化|持久化]]：RDB 快照、AOF 日志、混合持久化
- [[web后端开发/数据库/Redis/过期与内存淘汰|过期与内存淘汰]]：惰性/定期删除、八种淘汰策略
- [[web后端开发/数据库/Redis/事务与Lua|事务与Lua]]：MULTI/EXEC/WATCH、Pipeline、Lua 脚本
- [[web后端开发/数据库/Redis/主从与集群|主从与集群]]：主从复制原理、哨兵、分片集群

## 实战

- [[web后端开发/数据库/Redis/缓存|缓存]]：旁路缓存、缓存一致性、穿透/击穿/雪崩
- [[web后端开发/数据库/Redis/分布式锁|分布式锁]]：SET NX EX、Redisson、看门狗
- [[web后端开发/数据库/Redis/面试题目|面试题目]]
