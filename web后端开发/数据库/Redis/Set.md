---
title: Set
source: https://redis.io/docs/latest/develop/data-types/sets/
created: 2026-09-10
updated: 2026-09-10
tags:
  - 数据库
---

# Set 类型

## 简介

Set 是**无序、不可重复**的 String 集合（重复添加自动去重），独有优势：

- 支持集合运算：**交集、并集、差集**。
- 支持随机弹出 / 随机取成员。
- 判断成员是否存在 O(1)。

## 常用命令

| 命令 | 示例 | 说明 |
| ---- | ---- | ---- |
| SADD key m1 [m2 ...] | `SADD like:1001 u1 u2` | 添加成员（重复无效） |
| SREM key m1 [m2 ...] | | 移除成员 |
| SMEMBERS key | | 所有成员 |
| SCARD key | | 成员数量 |
| SISMEMBER key member | `SISMEMBER like:1001 u1` | 是否存在（O(1)） |
| SPOP key [n] | | 随机弹出 n 个（删除） |
| SRANDMEMBER key [n] | | 随机返回 n 个（不删除） |
| SINTER key1 [key2 ...] | `SINTER follow:a follow:b` | **交集** |
| SUNION key1 [key2 ...] | | 并集 |
| SDIFF key1 [key2 ...] | `SDIFF follow:b follow:a` | **差集**（key1 有、key2 没有） |

## 使用场景

1. **点赞 / 收藏去重**：同一用户重复点赞不生效。
	```bash
	SADD like:1001 u1        # u1 点赞
	SCARD like:1001          # 点赞数
	SISMEMBER like:1001 u1   # 是否点过赞
	SREM like:1001 u1        # 取消点赞
	```
2. **共同关注 / 共同好友**：`SINTER follow:u1 follow:u2`。
3. **好友推荐**：`SDIFF follow:目标用户 follow:我` —— 他关注了而我没关注的人。
4. **抽奖**：
	- `SRANDMEMBER lottery 3`：随机抽 3 人，可重复中奖（抽完放回）。
	- `SPOP lottery 3`：随机抽 3 人并移除，一人最多一次。

## 底层编码（了解）

- **intset（整数集合）**：全部成员都是整数且数量 ≤ `set-max-intset-entries`（512）时使用，有序整数数组，非常省内存。
- **hashtable**：不满足条件时转为哈希表，保证 O(1) 读写。
- 7.2+ 对小集合也引入了 listpack 编码（了解即可）。
