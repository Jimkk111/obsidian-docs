---
title: String
source: https://redis.io/docs/latest/develop/data-types/strings/
created: 2026-09-10
updated: 2026-09-10
tags:
  - 数据库
---

# String 类型

## 简介

String 是最基本、最常用的类型：value 是一个字符串。根据内容可以细分为三类用法：

- **文本**：普通字符串、JSON 字符串。
- **数字**：整数 / 浮点数，可执行原子自增自减。
- **二进制**：图片、序列化字节等（单个 value 最大 **512MB**）。

## 常用命令

| 命令 | 示例 | 说明 |
| ---- | ---- | ---- |
| SET key value [选项] | `SET name rose` | 选项见下表 |
| GET key | `GET name` | 不存在返回 nil |
| MSET / MGET | `MSET k1 v1 k2 v2` | 批量设置 / 批量获取（减少网络往返） |
| SETNX key value | `SETNX lock 1` | 不存在才设置（存在则不生效） |
| SETEX key s value | `SETEX code 60 9527` | 设置并指定秒级过期 |
| INCR / DECR key | `INCR count` | 自增 / 自减 1 |
| INCRBY / DECRBY key n | `INCRBY count 5` | 自增 / 自减 n |
| INCRBYFLOAT key n | | 浮点数自增 |
| APPEND key value | | 追加到末尾 |
| STRLEN key | | 字符串长度 |
| GETRANGE / SETRANGE | `GETRANGE name 0 1` | 取 / 改子串 |

SET 常用选项（2.6.12+ 可组合，一条命令保证原子性）：

| 选项 | 说明 |
| ---- | ---- |
| EX seconds / PX ms | 设置过期时间 |
| NX | key 不存在才设置（**分布式锁的抢锁命令**） |
| XX | key 存在才设置 |
| KEEPTTL | 保留原有过期时间（7.0+） |

> `SET lock 1 NX EX 10` 等价于 `SETNX lock 1` + `EXPIRE lock 10`，但后者是两条命令、非原子，曾造成大量死锁事故——面试常考，详见 [[web后端开发/数据库/Redis/分布式锁|分布式锁]]。

## 使用场景

1. **缓存对象**：把对象序列化为 JSON 整体存取。
	```bash
	SET user:1 '{"id":1,"name":"rose","age":18}'
	```
	字段需要频繁单独修改时更适合用 Hash（见 [[web后端开发/数据库/Redis/Hash|Hash]]）。
2. **验证码 / 临时数据**：设置 TTL，到期自动删除。
	```bash
	SETEX code:13800001111 120 9527
	```
3. **计数器 / 全局唯一 ID**：INCR 是单线程原子操作，并发下不会丢计数。
	```bash
	INCR article:1001:views      # 文章浏览量 +1
	INCR id:order                # 订单号递增
	```
4. **分布式锁**：`SET lock xxx NX EX 10`，见 [[web后端开发/数据库/Redis/分布式锁|分布式锁]]。

## 底层编码（了解）

| 编码 | 条件 | 说明 |
| ---- | ---- | ---- |
| int | 值可转为 long 的整数 | 直接存 long |
| embstr | 长度 ≤ 44 字节 | RedisObject 与 SDS 连续分配一块内存，短字符串高效 |
| raw | 长度 > 44 字节 | RedisObject 与 SDS 分开分配，指针指向 SDS |

字符串底层是 **SDS（Simple Dynamic String）**，相比 C 原生字符串：

- 结构头记录长度，`STRLEN` O(1)。
- **二进制安全**：按长度读，可存任意字节（图片、JSON）。
- **空间预分配 + 惰性释放**：追加时预留空间、缩短时不立即回收，减少内存重分配。
