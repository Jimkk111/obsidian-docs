---
title: DCL
source: https://dev.mysql.com/doc/refman/8.0/en/
created: 2026-09-10
updated: 2026-09-10
tags:
  - 数据库
---

# DCL（数据控制语言）

DCL（Data Control Language）管理数据库的用户与权限。线上系统不允许应用直连 root，需要按最小权限原则创建专用账号。

MySQL 中账号由**用户名 + 主机名（host）**共同标识：`'user'@'host'`。host 决定该账号能在哪里连接，`'%'` 匹配任意主机，`'localhost'` 仅本机。

## 一、管理用户

用户信息存在内置的 `mysql` 库的 `user` 表中。

```sql
-- 创建用户（8.0 中 GRANT 不再隐式创建用户，必须先 CREATE USER）
CREATE USER 'dev'@'%' IDENTIFIED BY 'Dev@123456';

-- 修改密码
ALTER USER 'dev'@'%' IDENTIFIED BY 'NewPass@123';

-- 删除用户
DROP USER 'dev'@'%';

-- 查看用户（需要相应权限）
SELECT user, host FROM mysql.user;
```

## 二、权限控制

常用权限：`SELECT`、`INSERT`、`UPDATE`、`DELETE`、`CREATE`、`ALTER`、`DROP`；`ALL` / `ALL PRIVILEGES` 表示全部权限。

```sql
-- 授权：权限列表 ON 库.表 TO 用户
-- *.* 表示所有库所有表；db.* 表示某库所有表
GRANT SELECT, INSERT, UPDATE ON sky.* TO 'dev'@'%';

-- 查看权限
SHOW GRANTS FOR 'dev'@'%';

-- 撤销权限
REVOKE INSERT, UPDATE ON sky.* FROM 'dev'@'%';
```

- `GRANT ... WITH GRANT OPTION` 允许该用户把自己拥有的权限再授予他人。
- 权限修改对**新连接**生效，已建立连接的会话不受影响；可用 `FLUSH PRIVILEGES` 重新加载（CREATE/GRANT 等语句本身会自动刷新）。
