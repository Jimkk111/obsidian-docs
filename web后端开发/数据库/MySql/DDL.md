---
title: DDL
source: https://dev.mysql.com/doc/refman/8.0/en/
created: 2026-09-10
updated: 2026-09-10
tags:
  - 数据库
---

# DDL（数据定义语言）

DDL（Data Definition Language）用于定义和管理数据库对象的结构：数据库、表、字段、索引、视图等。官方手册 13.1 节（Data Definition Statements）。

> 注意：DDL 语句会造成**隐式提交**——执行前会自动提交当前事务，且 DDL 本身不能回滚。

## 一、数据库操作

```sql
-- 创建：指定名称已存在会报错，IF NOT EXISTS 可避免
CREATE DATABASE [IF NOT EXISTS] db_name
    [CHARACTER SET utf8mb4]
    [COLLATE utf8mb4_0900_ai_ci];

-- 查询所有数据库
SHOW DATABASES;

-- 切换/使用数据库
USE db_name;
SELECT DATABASE();          -- 查看当前所在数据库

-- 查看建库语句（可查看字符集等信息）
SHOW CREATE DATABASE db_name;

-- 修改（一般只改字符集和排序规则）
ALTER DATABASE db_name CHARACTER SET utf8mb4;

-- 删除
DROP DATABASE [IF EXISTS] db_name;
```

字符集建议：

- MySQL 8.0 默认字符集是 **utf8mb4**（排序规则 `utf8mb4_0900_ai_ci`），5.7 及以前默认 latin1。
- MySQL 中的 `utf8` 实为 `utf8mb3` 的别名，最多 3 字节，存不下 emoji 等 4 字节字符，**建库建表应使用 utf8mb4**。

## 二、表操作

### 1. 查看表

```sql
SHOW TABLES;                 -- 当前库中所有表
DESC tb_user;                -- 表结构（字段、类型、键、默认值等）
SHOW CREATE TABLE tb_user;   -- 建表语句
```

### 2. 创建表

```sql
CREATE TABLE [IF NOT EXISTS] tb_user (
    id          BIGINT       PRIMARY KEY AUTO_INCREMENT COMMENT '编号',
    username    VARCHAR(20)  NOT NULL UNIQUE           COMMENT '用户名',
    age         TINYINT UNSIGNED COMMENT '年龄',
    gender      CHAR(1)      DEFAULT '男'              COMMENT '性别',
    phone       CHAR(11)     COMMENT '手机号',
    created_at  DATETIME     DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COMMENT '用户表';
```

- 每个字段定义：`字段名 数据类型 [约束] [COMMENT '注释']`。
- 最后一列之后**不能有逗号**。
- 类型见 [[web后端开发/数据库/MySql/数据类型|数据类型]]，约束见下文。

### 3. 修改表

```sql
-- 加字段（FIRST / AFTER 指定位置，默认追加到末尾）
ALTER TABLE tb_user ADD COLUMN nickname VARCHAR(30) AFTER username;

-- 修改字段类型（不能改名；可改位置）
ALTER TABLE tb_user MODIFY COLUMN nickname VARCHAR(50);

-- 修改字段名和类型
ALTER TABLE tb_user CHANGE COLUMN nickname nick_name VARCHAR(50);

-- 仅改字段名（8.0 支持）
ALTER TABLE tb_user RENAME COLUMN nick_name TO nickname;

-- 删除字段
ALTER TABLE tb_user DROP COLUMN nickname;

-- 修改表名
RENAME TABLE tb_user TO t_user;
ALTER TABLE t_user RENAME TO tb_user;
```

### 4. 删除表

```sql
DROP TABLE [IF EXISTS] tb_user;   -- 删除表结构和数据
TRUNCATE TABLE tb_user;           -- 清空全部数据，保留表结构（见 DML 中与 DELETE 的对比）
```

## 三、约束

约束（Constraint）作用于表中字段，用于限制存储的数据，保证数据的**完整性、正确性和一致性**。MySQL 8.0 中的六大约束：

| 约束 | 关键字 | 说明 |
| ---- | ---- | ---- |
| 非空约束 | `NOT NULL` | 字段值不能为 NULL |
| 唯一约束 | `UNIQUE` | 字段值全表唯一，NULL 不参与判重 |
| 主键约束 | `PRIMARY KEY` | 非空且唯一，一张表只能有一个 |
| 默认约束 | `DEFAULT` | 未指定值时使用默认值 |
| 检查约束 | `CHECK (条件)` | 8.0.16 起真正强制校验（之前仅解析不生效） |
| 外键约束 | `FOREIGN KEY` | 让两张表的数据建立关联，保证一致性 |

补充规则：

- 主键可以配合 `AUTO_INCREMENT` 让数值自动递增（要求该列必须是一个键）；一张表只能有一个自增列。
- **联合主键 / 联合唯一**必须写成表级约束：`PRIMARY KEY (a, b)`、`UNIQUE (a, b)`。
- 建表后追加/删除约束用 `ALTER TABLE ... ADD/DROP CONSTRAINT`，或通过 `MODIFY` 改写列定义实现。

```sql
CREATE TABLE dept (
    id   INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(30) NOT NULL
);

CREATE TABLE emp (
    id       INT PRIMARY KEY AUTO_INCREMENT,
    name     VARCHAR(30) NOT NULL,
    age      INT CHECK (age > 0 AND age <= 150),
    entrydate DATE,
    dept_id  INT,
    CONSTRAINT fk_emp_dept FOREIGN KEY (dept_id) REFERENCES dept(id)
        ON UPDATE CASCADE ON DELETE RESTRICT
);
```

### 外键的删除/更新行为（reference_option）

| 选项 | 行为 |
| ---- | ---- |
| `RESTRICT` / `NO ACTION`（默认） | 父表有关联记录时，拒绝删除/更新 |
| `CASCADE` | 父表删除/更新时，子表关联记录跟着删除/更新 |
| `SET NULL` | 父表删除/更新时，子表关联字段置为 NULL（要求该字段允许 NULL） |
| `SET DEFAULT` | 置为默认值（InnoDB 不支持） |

> 物理外键会影响性能，互联网业务中通常不建外键，而是在应用层保证一致性；但必须理解其语义。

## 四、索引的 DDL

索引是加速查询的数据结构（InnoDB 中为 B+ 树），其创建与删除属于 DDL：

```sql
CREATE [UNIQUE | FULLTEXT | SPATIAL] INDEX idx_name ON tb_user(phone);
DROP INDEX idx_name ON tb_user;
```

> 索引的原理、结构与失效场景见 [[web后端开发/数据库/MySql/索引|索引]]。

## 五、视图的 DDL

视图（View）是一条 SELECT 语句执行结果的命名包装，本身不存数据（简化查询、控制权限）：

```sql
CREATE VIEW v_user_20 AS SELECT id, username FROM tb_user WHERE age > 20;
SELECT * FROM v_user_20;
DROP VIEW [IF EXISTS] v_user_20;
```
