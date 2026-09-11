---
title: List
source: https://redis.io/docs/latest/develop/data-types/lists/
created: 2026-09-10
updated: 2026-09-10
tags:
  - 数据库
---

# List 类型

## 简介

List 是**双向链表**结构：

- **有序**（按插入顺序）、**元素可重复**。
- 两端（左/右）都能插入和弹出 → 可以当**栈**、**队列**使用。
- 支持按索引访问（LRANGE / LINDEX）。

## 常用命令

| 命令 | 示例 | 说明 |
| ---- | ---- | ---- |
| LPUSH key v1 [v2 ...] | `LPUSH msgs a b c` | 从**左**插入（可多个） |
| RPUSH key v1 [v2 ...] | `RPUSH msgs a b c` | 从**右**插入 |
| LPOP / RPOP key [n] | `RPOP msgs` | 从左 / 右弹出并删除 |
| LRANGE key start stop | `LRANGE msgs 0 -1` | 按索引取片段（`0 -1` 即全部，不删除） |
| LINDEX key index | | 取指定索引元素 |
| LLEN key | | 长度 |
| BLPOP / BRPOP key timeout | `BRPOP msgs 5` | 没有元素时**阻塞等待**；`0` 表示一直等（单位秒） |

```bash
127.0.0.1:6379> LPUSH queue t1 t2 t3      # 左插：链表为 t3 t2 t1
(integer) 3
127.0.0.1:6379> RPOP queue                # 右出 → 先进先出（队列）
"t1"
```

## 使用场景

1. **简单消息队列**：生产者 `LPUSH`，消费者 `BRPOP` 阻塞消费（先进先出 + 阻塞省轮询）。
	> List 做队列的局限：① 消息取出即删除，**消费失败无法重试**（无 ACK 机制）② 一条消息只能被一个消费者取走，**无法广播**。需要可靠队列用 [[web后端开发/数据库/Redis/其他类型|Stream]]。
2. **最新列表**：朋友圈时间线、最新评论 / 最新登录日志。
	```bash
	LPUSH moments:1001 "内容..."     # 发布
	LRANGE moments:1001 0 4         # 取最新 5 条
	```
3. **栈**：`LPUSH` + `LPOP`；**队列**：`LPUSH` + `RPOP`。

## 底层编码（了解）

quicklist（3.2+）：由多个 **listpack** 节点（7.0 前为 ziplist）组成的双向链表——每个节点是一小段连续内存的紧凑列表，节点之间双向串接。这样既保留链表两端插入删除 O(1) 的优势，又通过小段连续内存减少大链表的指针开销与内存碎片。
