---
title: SSH
source: https://www.yuque.com/mook-mvqcp/gvuey6/rpolmu5517ygzre2
created: 2026-08-15
updated: 2026-09-02
tags:
  - 计算机网络
---

**生成密钥**

ssh-keygen

**shh连接**

ssh

如何表示用户、目标服务器？user@servername，servername后面就是path。其实就是标准url的一部分。

互联网标准中，完整的url表示为：协议://[用户名:密码@]主机名[:端口号]/路径。

**配置文件**

```plain

Host servername #这里表示要连接的服务器的别名，在ssh -T user@servername的时候使用。作用是在本地配置索引用。
  HostName ip/dn #要连接的服务器的真实地址或者域名
  User user #作为什么用户连接目标服务器
  Port port #端口
  IdentityFile ~/.ssh/id_rsa #指定连接目标服务器所需的密钥文件

```
