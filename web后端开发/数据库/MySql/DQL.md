---
title: DQL
source: https://dev.mysql.com/doc/refman/8.0/en/select.html
created: 2026-09-10
updated: 2026-09-10
tags:
  - 数据库
---

# DQL（数据查询语言）

DQL（Data Query Language）的核心是 `SELECT` 语句，从表中查找数据并返回（官方手册 13.2.10）。多表查询见 [[web后端开发/数据库/MySql/多表查询|多表查询]]，函数见 [[web后端开发/数据库/MySql/函数|函数]]。

## 一、完整语法与执行顺序

```sql
SELECT      [DISTINCT] 字段列表
FROM        表名列表
WHERE       条件列表
GROUP BY    分组字段列表
HAVING      分组后条件列表
ORDER BY    排序字段列表
LIMIT       起始索引, 查询记录数;
```

书写顺序如上，而**逻辑执行顺序**为：

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

因此 WHERE 中不能用 SELECT 里起的别名（此时别名还没生成），而 ORDER BY 可以。

## 二、基础查询

```sql
-- 查询指定列（多列逗号分隔，最后一列后不能有逗号）
SELECT username, age FROM tb_user;

-- 查询所有列（* 会降低可读性，生产环境不建议）
SELECT * FROM tb_user;

-- 起别名（AS 可省略），用于美化列名或区分多表同名字段
SELECT username AS 用户名, age 年龄 FROM tb_user;

-- 去重
SELECT DISTINCT gender FROM tb_user;
```

## 三、条件查询（WHERE）

### 1. 比较与范围运算符

| 运算符 | 说明 |
| ---- | ---- |
| `=`、`<>`（或 `!=`）、`>`、`<`、`>=`、`<=` | 比较运算，`<>` 为不等于 |
| `BETWEEN ... AND ...` | 在某闭区间内（两个边界值均包含） |
| `IN (v1, v2, ...)` | 在指定**集合**中取值（与 BETWEEN 的连续范围不同） |
| `LIKE` / `NOT LIKE` | 模糊匹配：`%` 匹配任意个字符（含0个），`_` 匹配单个字符 |
| `IS NULL` / `IS NOT NULL` | 判空。NULL 与任何值用 `=` 比较结果都是 NULL，**必须用 IS NULL** |

```sql
SELECT * FROM emp WHERE age >= 20 AND age <= 30;
SELECT * FROM emp WHERE age BETWEEN 20 AND 30;   -- 等价于上一句
SELECT * FROM emp WHERE dept_id IN (1, 3);       -- 等价于 dept_id = 1 OR dept_id = 3
SELECT * FROM emp WHERE username LIKE '张%';     -- 姓张的
SELECT * FROM emp WHERE username LIKE '_三';     -- 两个字符且第二个是三
SELECT * FROM emp WHERE phone IS NULL;
```

### 2. 逻辑运算符

| 运算符 | 说明 |
| ---- | ---- |
| `AND`（`&&`） | 并且，多个条件同时成立 |
| `OR`（`||`） | 或者，任一条件成立 |
| `NOT`（`!`） | 取反，排除满足条件的行 |

`AND` 优先级高于 `OR`；需要改变优先级时用 `()` 将条件包裹。`&&`、`||`、`!` 是 MySQL 扩展，标准写法是单词形式。

## 四、聚合函数

聚合函数（aggregate function）将一列或多行数据作为一个整体进行纵向计算，**NULL 值不参与聚合运算**（`COUNT(*)` 除外）。

| 函数 | 说明 |
| ---- | ---- |
| COUNT(expr) | 统计数量 |
| SUM(expr) | 求和 |
| AVG(expr) | 平均值 |
| MAX(expr) | 最大值 |
| MIN(expr) | 最小值 |

```sql
SELECT COUNT(*) FROM emp;                    -- 统计总行数
SELECT AVG(age) FROM emp;                    -- 平均年龄
SELECT MAX(age), MIN(age) FROM emp;
```

- `COUNT(*)` 统计行数；`COUNT(字段)` 统计该字段非 NULL 的行数；InnoDB 下 `COUNT(*)` 与 `COUNT(1)` 性能相当。
- 聚合查询的结果是单行，**不能与普通字段混着 SELECT**。

## 五、分组查询（GROUP BY）

```sql
SELECT gender, COUNT(*) AS 人数
FROM emp
WHERE age < 60          -- 分组前过滤
GROUP BY gender
HAVING COUNT(*) > 10;   -- 分组后过滤
```

- 按指定字段分组后，SELECT 的是分组字段和聚合结果。
- MySQL 8.0 默认开启 `ONLY_FULL_GROUP_BY` 模式：SELECT 中出现的非聚合字段必须出现在 GROUP BY 中（或函数依赖于分组字段），否则报错。
- **WHERE 与 HAVING 的区别**：WHERE 在分组前过滤，不能使用聚合函数；HAVING 在分组后对分组结果过滤，可以使用聚合函数。
- `GROUP BY ... WITH ROLLUP` 会在结果末尾追加一行对所有分组的汇总。

## 六、排序查询（ORDER BY）

```sql
-- ASC 升序（默认），DESC 降序
SELECT * FROM emp ORDER BY age DESC;

-- 多字段排序：先按第一个字段排，值相同时再按第二个字段排，依此类推
SELECT * FROM emp ORDER BY age ASC, entrydate DESC;
```

- 默认升序 ASC；DESC 表示降序。
- 排序字段出现 NULL 时：升序 NULL 在最前，降序 NULL 在最后。

## 七、分页查询（LIMIT）

```sql
-- 返回前 10 条
SELECT * FROM emp LIMIT 10;

-- 从第 6 条开始（索引从 0 计），返回 10 条
SELECT * FROM emp LIMIT 5, 10;
SELECT * FROM emp LIMIT 10 OFFSET 5;   -- 等价写法：LIMIT 行数 OFFSET 起始索引
```

- 起始索引从 0 开始计算：`起始索引 = (页码 - 1) * 每页记录数`。
- 第 1 页每页 10 条：`LIMIT 0, 10`；第 2 页：`LIMIT 10, 10`。
- LIMIT 必须是整数量，不能是表达式变量；深分页（`LIMIT 1000000, 10`）性能差，需要优化。
