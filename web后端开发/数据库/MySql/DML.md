---
title: DML
source: https://dev.mysql.com/doc/refman/8.0/en/
created: 2026-09-10
updated: 2026-09-10
tags:
  - 数据库
---

# DML（数据操纵语言）

DML（Data Manipulation Language）用于对表中的**数据**进行增、删、改，即 `INSERT`、`UPDATE`、`DELETE`（官方手册中 `SELECT` 也属于 DML，本笔记单独放在 [[web后端开发/数据库/MySql/DQL|DQL]]）。

## 一、INSERT 添加数据

```sql
-- 为指定字段添加一条记录（推荐：显式列出字段，表结构变化时不易错位）
INSERT INTO tb_user (id, username, age, gender)
VALUES (1, '张三', 25, '男');

-- 同时插入多条记录
INSERT INTO tb_user (username, age) VALUES
    ('李四', 30),
    ('王五', 28);

-- 为全部字段添加（必须按建表字段顺序给全，不推荐）
INSERT INTO tb_user VALUES (4, '赵六', 22, '女', '13800000004', NOW());
```

- 字段与值的个数、顺序、类型要一一对应；字符串和日期用引号包裹。
- 插入数据时，自增列可以不写（自动生成），有默认值的列不写则取默认值。
- `INSERT ... SET col = val, ...` 是 MySQL 扩展写法。

## 二、UPDATE 修改数据

```sql
UPDATE tb_user
SET age = 26, username = '张三丰'
WHERE id = 1;
```

- **不带 WHERE 条件会修改整张表的所有记录**，更新前建议先用同条件的 SELECT 验证范围。
- MySQL 支持 `ORDER BY` / `LIMIT` 限定更新范围（多线程复制下该特性不安全）。
- 多表更新：`UPDATE t1 JOIN t2 ON ... SET t1.c = t2.c WHERE ...`。

## 三、DELETE 删除数据

```sql
-- 按条件删除
DELETE FROM tb_user WHERE id = 1;

-- 删除整表数据（逐行删除，不重置自增计数器）
DELETE FROM tb_user;
```

## 四、TRUNCATE 与 DELETE 的区别

两者都能清空表数据，但有本质差异：

| 对比项 | DELETE | TRUNCATE TABLE |
| ---- | ---- | ---- |
| 分类 | DML，可加 WHERE | DDL，不能加 WHERE |
| 实现方式 | 逐行删除 | 删除表并按原结构重建 |
| 事务 | 事务安全，**可以回滚** | **不能回滚**（隐式提交） |
| 自增值 | 不重置 | 重置为初始值 |
| 触发器 | 触发 ON DELETE 触发器 | 不触发 |
| 性能 | 大表较慢 | 快 |

## 五、UPSERT（冲突时更新）

MySQL 提供两种「存在则更新、不存在则插入」的写法，依赖主键或唯一约束判重：

```sql
-- 存在唯一键冲突时改为更新指定字段
INSERT INTO tb_user (id, username, age) VALUES (1, '张三', 26)
ON DUPLICATE KEY UPDATE age = 26;

-- REPLACE：冲突时先删除旧行再插入新行（会改变主键值、触发删除+插入）
REPLACE INTO tb_user (id, username, age) VALUES (1, '张三', 26);
```
