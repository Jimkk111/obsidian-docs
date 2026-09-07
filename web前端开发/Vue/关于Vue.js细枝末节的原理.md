---
title: 关于Vue.js细枝末节的原理
source: https://www.yuque.com/mook-mvqcp/wc9wsu/lvg5ebxn8zxf2r07
created: 2026-04-18
updated: 2026-04-18
tags:
  - Vue
---

**1.$*属性是什么？**

带有$前缀的属性是全局属性，这样做的目的：一是有标识作用、二是避免命名冲突。

**2.在模板使用ref响应式对象**

在模板使用ref响应式对象不需要加.value，因为Vue模板编译器会解包，自动读取这个ref对象的value属性。
