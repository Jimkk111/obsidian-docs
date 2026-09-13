---
title: uniapp应用开发
created: 2026-09-13
updated: 2026-09-13
tags:
  - MOC
---

# uniapp应用开发

依据 uni-app 官方文档（uniapp.dcloud.net.cn）框架 / 生命周期 / 路由 / 组件 / API / 条件编译等板块组织的教程笔记，按阅读顺序排列，并附实战项目（瑞吉外卖小程序）归纳的 Web 差异对照。

- [[小程序开发/uniapp应用开发/跨端原理与工程结构|跨端原理与工程结构]]——编译器+运行时两层模型、HBuilderX/CLI 两种工程、pages.json/manifest
- [[小程序开发/uniapp应用开发/生命周期体系|生命周期体系]]——应用/页面/组件三套钩子叠加，onLoad/onShow 的实战语义
- [[小程序开发/uniapp应用开发/路由与页面栈|路由与页面栈]]——uni.* 导航 API、10 层栈、EventChannel、与 vue-router 对照
- [[小程序开发/uniapp应用开发/组件-easycom与样式适配|组件：easycom 与样式适配]]——小程序组件规范、easycom、rpx/uni.scss/样式穿透/安全区
- [[小程序开发/uniapp应用开发/API体系与条件编译|API 体系与条件编译]]——uni.* 全家桶、请求层封装范式、#ifdef、Pinia 接入
- [[小程序开发/uniapp应用开发/微信登录链路与支付|微信登录链路与支付]]——uni.login→code2Session、头像昵称填写能力、requestPayment 责任划分
- [[小程序开发/uniapp应用开发/Web开发差异对照表|Web 开发差异对照表]]——给 Web 开发者的迁移速查（含实战项目映射）

上级：[[小程序开发/小程序开发|小程序开发]]

> 本目录笔记仅第 2/7/10/11/12 篇处于全局主线中，跨目录的推荐阅读顺序见 [[小程序开发/小程序开发|小程序开发 MOC]]；每篇笔记末尾附阅读导航可前后跳转。
