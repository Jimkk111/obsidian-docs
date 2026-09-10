---
title: 事务与Lua
source:
  - https://redis.io/docs/latest/develop/interact/transactions/
  - https://redis.io/docs/latest/commands/eval/
created: 2026-09-10
updated: 2026-09-10
tags:
  - 数据库
---

# 事务、Pipeline 与 Lua

## Redis 事务

三个阶段：**MULTI 开启 → 命令依次入队（返回 QUEUED）→ EXEC 原子执行**（DISCARD 放弃事务）。

```bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> SET k1 v1
QUEUED
127.0.0.1:6379(TX)> INCR cnt
QUEUED
127.0.0.1:6379(TX)> EXEC
1) OK
2) (integer) 1
```

- EXEC 之前命令只入队**不执行**。
- **入队时报错**（命令不存在 / 参数个数错误等）：EXEC 时**整个事务拒绝执行**。
- **运行时报错**（如对 String 执行 LPUSH）：只有出错的那条返回错误，**其余命令照常执行，不回滚**。

### 为什么不支持回滚

官方解释：命令失败只应来自编程错误（应在开发/测试阶段发现），不支持回滚可以让 Redis 内部保持**简单、快速、无锁**。这与 MySQL 完全不同：

| | Redis 事务 | MySQL 事务（InnoDB） |
| ---- | ---- | ---- |
| 原子性 | 部分：执行期间不被其他客户端插队，但**出错不回滚** | 完整（出错回滚） |
| 隔离性 | 事务执行期间不被其他客户端命令插入 | 多种隔离级别（见 [[web后端开发/数据库/MySql/事务|事务]]） |
| 一致性 / 持久性 | 依赖应用与持久化机制 | redo / undo log 保证 |

### WATCH 乐观锁

`WATCH key` 后进入事务：EXEC 时若发现被 WATCH 的 key 已被**其他客户端修改**，整个事务放弃执行（EXEC 返回 nil）——类似 CAS 的版本比对。`UNWATCH` 取消监视。

```bash
WATCH balance     # 监视余额
MULTI
DECRBY balance 100
EXEC             # 若期间别人改过 balance，这里返回 nil，应用层决定是否重试
```

## Pipeline（管道）

- 客户端把一批命令**打包后一次发送**，服务端批量返回结果，减少**网络往返（RTT）**：100 条命令从 100 次 RTT 降为 1 次。
- 只是网络层优化，**不保证原子性**，命令之间可能插入其他客户端请求。

| | 事务 | Pipeline |
| ---- | ---- | ---- |
| 目的 | 命令打包、执行不被插队 | 减少网络 IO |
| 原子性 | 是（无插队） | 否 |
| 关系 | 二者可结合：在管道中提交事务 | |

## Lua 脚本

Redis 内置 Lua 解释器，脚本内通过 `redis.call` 执行命令：

```bash
# EVAL "脚本" numkeys key... arg...
EVAL "return redis.call('set', KEYS[1], ARGV[1])" 1 lock uuid-1
```

- 脚本执行期间 Redis **阻塞其他命令**（单线程），因此天然**原子**——「读取 → 判断 → 写入」多步合一。
- 典型场景：**分布式锁的释放**（判断是自己的锁才删）、限流、库存扣减。
- 注意：脚本要短小避免阻塞；用 `SCRIPT LOAD` + `EVALSHA` 可复用脚本省网络。

```lua
-- 释放锁：只有锁的持有者才能删除（对应 [[web后端开发/数据库/Redis/分布式锁|分布式锁]] 的误删问题）
if redis.call('get', KEYS[1]) == ARGV[1] then
    return redis.call('del', KEYS[1])
else
    return 0
end
```
