---
title: MVCC
source: https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html
created: 2026-09-10
updated: 2026-09-11
tags:
  - 数据库
---

# MVCC（多版本并发控制）

MVCC（Multi-Version Concurrency Control）是 InnoDB 在 **READ COMMITTED 和 REPEATABLE READ** 隔离级别下，通过「数据多版本 + 快照读」实现的**非锁定一致性读**（官方 15.7.2.3 Consistent Nonlocking Reads）：读操作不加锁，读写互不阻塞，大幅提升并发性能。隔离级别的背景见 [[web后端开发/数据库/MySql/事务|事务]]。

MVCC 由三件套实现：**隐藏字段 + undo log 版本链 + ReadView**。

## 一、隐藏字段

InnoDB 每行记录都带三个隐藏列：

| 字段 | 说明 |
| ---- | ---- |
| DB_TRX_ID | 最近一次修改（插入/更新/删除）该行的事务 ID |
| DB_ROLL_PTR | 回滚指针，指向该行的上一版本（undo log 中） |
| DB_ROW_ID | 隐藏主键（6 字节），表中无主键且无非空唯一索引时才使用 |

## 二、undo log 版本链

每次修改数据时，旧版本被写入 undo log，通过 `DB_ROLL_PTR` 把各个历史版本串成**从新到旧**的链表：

```
当前行 (trx_id=300, name='C')
   └─ roll_ptr → 版本2 (trx_id=200, name='B')
                    └─ roll_ptr → 版本1 (trx_id=100, name='A')
```

一行数据同时存在多个版本，不同事务按各自的可见性规则读取不同版本——这就是「多版本」。undo log 同时服务于事务回滚（原子性）和 MVCC。

## 三、ReadView（读视图）

事务执行快照读时会生成一个 ReadView，包含四个关键信息：

| 字段 | 含义 |
| ---- | ---- |
| m_ids | 生成 ReadView 时**活跃（未提交）**事务的 ID 集合 |
| min_trx_id | m_ids 中最小的事务 ID |
| max_trx_id | 系统中**下一个将要分配**的事务 ID（不是 m_ids 的最大值） |
| creator_trx_id | 创建该 ReadView 的事务自己的 ID |

对版本链上每个版本，用其 `trx_id` 判断可见性：

1. `trx_id == creator_trx_id`：自己修改的，**可见**。
2. `trx_id < min_trx_id`：生成视图前已提交，**可见**。
3. `trx_id >= max_trx_id`：生成视图之后才开启的事务，**不可见**。
4. `min_trx_id <= trx_id < max_trx_id`：在 m_ids 中（当时还活跃）→ **不可见**；不在 m_ids 中（已提交）→ **可见**。

不可见时，沿 `DB_ROLL_PTR` 找上一个版本继续判断，直到找到可见版本或链尽头（返回空）。

## 四、RC 与 RR 的区别：ReadView 的生成时机

这是两种隔离级别行为差异的根源：

| 隔离级别 | ReadView 生成时机 | 效果 |
| ---- | ---- | ---- |
| READ COMMITTED | **每次** SELECT 都生成新的 | 能读到别的事务刚提交的数据 → 不可重复读 |
| REPEATABLE READ | 仅**第一次** SELECT 时生成，之后复用 | 事务内多次读结果一致 → 可重复读 |

## 五、快照读与当前读

| 类型 | 语句 | 特点 |
| ---- | ---- | ---- |
| 快照读 | 普通 `SELECT` | 读 MVCC 版本链，不加锁 |
| 当前读 | `INSERT` / `UPDATE` / `DELETE`；`SELECT ... FOR UPDATE`；`SELECT ... FOR SHARE` | 读**最新已提交版本**并加锁（见 [[web后端开发/数据库/MySql/锁\|锁]]） |

增删改必须基于最新数据，因此都是当前读。

## 六、小结

- MVCC 解决的是**并发读写**的隔离问题：普通 SELECT 不加锁也能看到一致的数据。
- RR 级别下 MVCC + undo log 版本链解决了**脏读、不可重复读**；对幻读，快照读靠 MVCC 看不到新插入的行，当前读靠 **next-key lock** 阻止范围内插入（见 [[web后端开发/数据库/MySql/锁|锁]]）。
- 长事务会阻止旧版本 undo log 的清理，导致版本链变长、undo 表空间膨胀，应避免长事务。

### 2026-09-11 ReadView 可见性判断走查示例

用一个具体例子把上文三、四节的规则走一遍。版本链：

```text
当前行  (trx_id=300, name='C')            ← 事务300 改的，尚未提交
   └─→ undo 版本2 (trx_id=200, name='B')   ← 事务200 改的，已提交
          └─→ undo 版本1 (trx_id=100, name='A') ← 事务100 插入，已提交
```

事务 400（我自己）执行普通 `SELECT name`，生成 ReadView：

- `m_ids = {300, 400}`（300 未提交仍活跃；自己 400 也在集合中）
- `min_trx_id = 300`，`max_trx_id = 500`（系统下一个要分配的 id），`creator_trx_id = 400`

逐版本判断：

1. 当前行 trx_id=300：落在 min~max 之间且**在 m_ids 中** → 拍照时活跃、未提交 → **不可见**，顺 `DB_ROLL_PTR` 回退；
2. 版本2 trx_id=200：`200 < min_trx_id(300)` → 拍照前已提交 → **可见** → 返回 `'B'`。

结论：磁盘当前行明明是 `'C'`（未提交），我读到的是 `'B'`——全程不加锁、不阻塞事务 300，脏读被防住。这就是"多版本"的含义：**同一行数据，对不同快照的事务呈现出不同的值**。

一句话定义 MVCC：隐藏字段（版本编号）+ undo 版本链（历史仓库）+ ReadView（可见性规则），三件套让每个读事务"穿越回自己快照那一刻"去读数据——用空间（多存版本）换并发（读写不互斥）。
