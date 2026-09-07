---
title: Web漏洞
source: https://www.yuque.com/mook-mvqcp/wc9wsu/nke5i2cbg7sg1t70
created: 2026-05-24
updated: 2026-05-24
tags:
  - 计算机网络
---

# 一、跨站伪造请求CSRF

## （一）根本原因

CSRF的根本原因是浏览器允许跨站请求自动添加cookie，并且满足以下条件：

1. **服务端仅依靠 Cookie 验证身份**，没有额外检查请求的来源或携带 CSRF Token。

2. **攻击者能够构造一个跨站的恶意请求**（如通过表单、图片、脚本等）。

但是因为历史兼容性和合法的跨站需求，浏览器不能完全禁止跨站请求自动携带cookie，但提供了一个限制手段SameSite这个cookie属性来改变默认策略。

## （二）防范手段

# 二、跨站脚本攻击XSS
