---
title: Hash
source: https://redis.io/docs/latest/develop/data-types/hashes/
created: 2026-09-10
updated: 2026-09-10
tags:
  - 数据库
---

# Hash 类型

## 简介

Hash 的 value 本身是一张 **field-value（字段-值）哈希表**，类似 Java 的 `Map<String, Map<String, Object>>`，非常适合存对象：可以对单个字段独立读写，而不必动整个对象。

与 String 存对象（JSON）对比：

| 方式 | 优点 | 缺点 |
| ---- | ---- | ---- |
| String + JSON | 整存整取简单，内存占用略小 | 改一个字段也要整体序列化重写 |
| Hash | 字段级读写（HGET/HSET），字段更新高效 | 字段多时命令略繁琐 |

## 常用命令

| 命令 | 示例 | 说明 |
| ---- | ---- | ---- |
| HSET key field value [field value ...] | `HSET user:1 name rose age 18` | 设置一个或多个字段 |
| HGET key field | `HGET user:1 name` | 取单个字段 |
| HMGET key f1 [f2 ...] | `HMGET user:1 name age` | 取多个字段 |
| HGETALL key | | 取所有字段和值 |
| HDEL key f1 [f2 ...] | | 删除字段 |
| HLEN key | | 字段数量 |
| HEXISTS key field | | 字段是否存在 |
| HKEYS / HVALS key | | 所有字段 / 所有值 |
| HINCRBY key field n | `HINCRBY user:1 age 1` | 字段值自增 n（原子） |

> 7.4+ 支持 `HEXPIRE` 给单个字段设置过期时间（了解即可）。

## 使用场景

**购物车**（用户 1001，商品 10088）：

```bash
HSET cart:1001 10088 1          # 商品 10088 加入购物车 ×1
HINCRBY cart:1001 10088 1       # 再加一件
HINCRBY cart:1001 10088 -1      # 减一件（减到 0 可 HDEL）
HLEN cart:1001                  # 商品种类数
HGETALL cart:1001               # 购物车明细
HDEL cart:1001 10088            # 移除商品
```

其他：对象的部分字段缓存（如 `user:1` 的 name、balance 单独更新）、结合 Java 客户端见 [[web后端开发/数据库/Redis/Java客户端|Java客户端]]。

## 底层编码（了解）

- **listpack**（7.0 前为 ziplist 压缩列表）：元素少时的紧凑编码——连续内存按 `[field1][value1][field2][value2]...` 顺序存储，省内存但查找 O(n)。条件：字段数 ≤ `hash-max-listpack-entries`（128）且单个值 ≤ `hash-max-listpack-value`（64 字节）。
- **hashtable**：超过阈值后转为真正的哈希表，读写 O(1)。

小对象用紧凑结构、大对象用哈希表，是 Redis 各类型编码转换的统一套路（见 [[web后端开发/数据库/Redis/ZSet|ZSet]]、[[web后端开发/数据库/Redis/Set|Set]] 的编码说明）。
