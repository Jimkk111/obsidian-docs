---
title: web鉴权方式
source: https://www.yuque.com/mook-mvqcp/wc9wsu/yoro2lrzweyfta6f
created: 2026-05-28
updated: 2026-05-28
tags:
  - 计算机网络
---

### 1.HTTP Basic Authentication（基本认证）

### 2.Session/Cookie

首先弄明白什么是Session（会话）。session是服务器为每一位登录用户创建的一个数据对象，存储用户信息（用户id、角色、权限等），通常保存在内存中、redis或数据库中，每个session有一给唯一标识sessionId。

cookie是浏览器存储在本地中的小型文本数据，服务器通过Set-Cookie响应头将SessionID发送给浏览器，浏览器在后续请求自动将其携带在Cookie请求头中。

优点是简单易用、实时撤销、容量不受限等；缺点是有CSRF风险、资源占用、跨域限制等。
