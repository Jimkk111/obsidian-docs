---
title: ZSet
source: https://redis.io/docs/latest/develop/data-types/sorted-sets/
created: 2026-09-10
updated: 2026-09-10
tags:
  - 数据库
---

# ZSet（SortedSet）类型

## 简介

ZSet 是**可排序的 Set**：member 唯一，每个 member 关联一个 **score（分值，double 类型）**，集合按 score 有序排列。排序依据由自己指定（销量、分数、时间戳……），是**排行榜类需求的最佳选择**。

| 与 Set 的区别 | Set | ZSet |
| ---- | ---- | ---- |
| 重复 | 不可重复 | member 不可重复 |
| 排序 | 无序 | 按 score 有序 |
| 额外能力 | 交并差 | 排名、分数区间、分值加减 |

## 常用命令

| 命令 | 示例 | 说明 |
| ---- | ---- | ---- |
| ZADD key score member [...] | `ZADD rank 100 商品A` | 添加 / 更新成员及分数 |
| ZREM key member | | 删除成员 |
| ZSCORE key member | | 查分数 |
| ZINCRBY key n member | `ZINCRBY rank 1 商品A` | 分数原子加减 |
| ZCARD key | | 成员数量 |
| ZCOUNT key min max | | score 区间内的成员数量 |
| ZRANGE key start stop [WITHSCORES] | `ZRANGE rank 0 -1 WITHSCORES` | 按 score **升序**取排名区间 |
| ZREVRANGE key start stop [WITHSCORES] | `ZREVRANGE rank 0 9 WITHSCORES` | 按 score **降序**取（Top N） |
| ZRANGEBYSCORE key min max | `ZRANGEBYSCORE rank 80 (100` | score 区间成员；`(80` 排除边界，支持 `+inf/-inf` |
| ZRANK / ZREVRANK key member | | 升序 / 降序排名（从 0 开始） |

> 6.2+ 的 `ZRANGE` 支持 `BYSCORE`、`REV`、`LIMIT` 参数，可替代 ZRANGEBYSCORE / ZREVRANGE，新代码推荐统一用 ZRANGE。

## 使用场景

1. **排行榜**（销量榜、积分榜、热搜）：
	```bash
	ZINCRBY rank:2026-09-10 1 商品A              # 卖出一件 +1
	ZREVRANGE rank:2026-09-10 0 9 WITHSCORES     # 今日 Top10（降序带分数）
	ZREVRANK rank:2026-09-10 商品A               # 商品A 的排名
	ZSCORE rank:2026-09-10 商品A                  # 商品A 的销量
	```
2. **延迟队列**：score 存任务的执行时间戳，定时任务轮询到期任务。
	```bash
	ZADD delay:queue 1760000000 任务id           # 到期时间戳作 score
	ZRANGEBYSCORE delay:queue 0 <当前时间戳>     # 取出到期任务执行
	```
3. **优先级队列**、时间窗口限流（score 记时间戳）。

## 底层编码（了解）

- **listpack**（7.0 前为 ziplist）：小 ZSet —— 元素数 ≤ `zset-max-listpack-entries`（128）且单值 ≤ 64 字节，按 score 排序的紧凑连续内存。
- **skiplist 跳表 + dict 字典**：超过阈值转换。跳表按 score 有序（范围查询、排名 O(logN)）；字典存 member → score（查分 O(1)）。两套结构通过指针共享成员，空间代价可接受。

### 跳表为什么不用红黑树 / B+ 树（面试常问）

- 跳表：多层有序链表，每隔若干节点抽取到上层做索引，查找从顶层逐层下坠，O(logN)。**范围查询**沿底层链表顺序遍历天然友好；插入删除只改指针，实现远比红黑树简单。
- B+ 树是多路磁盘树，为「按页读磁盘」设计；Redis 纯内存无需页式结构。

> 面试扩展：ZSet 同时满足「按排名查询（跳表）」和「按 member 查 score（字典）」两种 O(logN)/O(1) 高效访问，这是双结构并存的理由。
