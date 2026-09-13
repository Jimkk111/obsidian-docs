---
title: Web 开发差异对照表
created: 2026-09-13
updated: 2026-09-13
source: 微信官方文档 + uni-app 官方文档 + 实战项目（瑞吉外卖小程序）归纳
tags:
  - 小程序开发
  - uniapp
  - 微信小程序
---

# Web 开发差异对照表

给 Web（Vue/React）开发者的小程序/uni-app 迁移速查。核心心智只有一句话：**从"操作浏览器"变成"调用平台能力"——没有 DOM/BOM，一切通过 uni./wx. API 与数据驱动。**

## 运行时

| Web                        | 小程序 / uni-app                                     | 影响面                                |
| -------------------------- | ------------------------------------------------- | ---------------------------------- |
| `document.querySelector`   | 不存在，数据驱动视图                                        | 所有 DOM 操作代码作废                      |
| `window.*` / `navigator.*` | `uni.getWindowInfo()` / `uni.getSystemInfoSync()` | 尺寸、平台、网络探测全换 API                   |
| `fetch` / `axios`（XHR）     | `uni.request`（小程序端无 CORS 概念）                      | 请求层必须重写封装                          |
| `localStorage`             | `uni.setStorageSync`（同步、单 key 1MB/总 10MB）         | 持久化方案改写                            |
| 事件循环同一 JS 环境               | 双线程：逻辑层与渲染层分离，setData 序列化传输                       | 性能模型完全不同                           |
| 依赖 DOM 的第三方库               | 不可用；找 uni 生态版或适配层                                 | echarts→ lime-echart/uni-ecanvas 等 |

## 模板与组件

| Web（Vue） | uni-app | 备注 |
|---|---|---|
| div / span / img | view / text / image | 文本必须包 text；image 必须定尺寸 |
| overflow 滚动 | scroll-view | 触底/触顶/scroll-into-view 都在这 |
| v-model 原生 input | 可用，但注意各端键盘/组合输入差异 | 微信端有 type="nickname" 特殊类型 |
| 组件手动 import 注册 | easycom 目录约定自动注册 | `components/x/x.vue` 免 import |
| Element/AntD | uni-ui（uni_modules + easycom） | 组件库生态不同 |
| CSS 任意选择器 | 支持子集（无通配符 `*`），组件默认样式隔离 | 穿透用 `:deep()` |

## 路由

| vue-router               | uni-app                     | 备注             |
| ------------------------ | --------------------------- | -------------- |
| 路由表 createRouter         | pages.json 配置式注册            | 页面必须注册才能访问     |
| router.push / RouterLink | `uni.navigateTo` 等 5 个 API  | 10 层页面栈上限      |
| params / 动态路由            | URL query + onLoad(options) | 参数全是字符串        |
| beforeEach 守卫            | 无；请求层 401 拦截 / 页面 onShow 自检 | 权限逻辑位置不同       |
| keep-alive               | 无概念，页面出栈前天然保留               | "返回刷新"靠 onShow |
| hash/history             | 仅 H5 端可配                    | 小程序无 URL 概念    |

## 生命周期

| Vue / Web | 页面级（uni-app） | 语义 |
|---|---|---|
| created/mounted | onLoad / onShow / onReady | onLoad 只一次且拿参数；onShow 每次进入 |
| beforeUnmount | onUnload | 页面销毁清理点 |
| —（依赖路由钩子） | onPullDownRefresh / onReachBottom | 下拉/触底是页面级能力 |
| visibilitychange | App.vue onShow/onHide | 应用级前后台切换 |

最大的实战差异：**mounted 只跑一次，从别的页面返回不会重跑**；"返回刷新"必须用 onShow + 标记位。

## 登录与平台能力

| Web | 小程序 | 备注 |
|---|---|---|
| 账号密码/OAuth 跳转 | `uni.login` 静默拿 code → 后端 code2Session 换登录态 | code 5 分钟一次性，openid 只在服务端 |
| 浏览器存储 session | 自定义登录态存 storage + 请求头注入 | 401 统一拦截处理 |
| 第三方支付页跳转 | `uni.requestPayment`（服务端签名） | 个人主体无真实支付 |
| alert/confirm | `uni.showToast` / `uni.showModal` | 无阻塞式弹窗 |
| 文件下载 + a 标签 | `uni.downloadFile` + `uni.openDocument` | 沙盒文件体系 |

## 调试与发布

| Web | 小程序 / uni-app |
|---|---|
| 浏览器 DevTools | 微信开发者工具（AppData/WXML 面板）+ HBuilderX 运行到各端 |
| npm run build 产物部署 | 编译产物（unpackage/dist）上传 → 体验版 → 审核 → 发布 |
| 随时可改随时生效 | 版本有审核周期，热更新受限；线上问题靠体验版回归 |
| 域名随意 | request 合法域名白名单（HTTPS+备案），开发期可勾选不校验 |

## 一个项目里的典型落地（瑞吉外卖小程序）

- `utils/request.js`：uni.request 封装 + Cookie 注入 + 401 处理（对应 Web 的 axios 拦截器）。
- `pages.json`：8 个页面注册 + 原生导航栏配色（对应 vue-router 路由表 + 布局配置）。
- `pages/index` 的 `onLoad(options)` 启动参数分支、`onShow` 免授权刷新：页面级生命周期实战。
- `uni.getMenuButtonBoundingClientRect()` 算导航高度：小程序特有布局题。
- `utils/webscoket.js`：`uni.connectSocket` + STOMP 订阅 RabbitMQ——Web 端 WebSocket 知识迁移到 uni API 的样本。

## 经验提示

- 写新代码时永远假设"没有 DOM"，H5 端才有的便利（如 window 事件）一律条件编译隔离。
- 平台能力（登录/支付/分享）先查目标端支持矩阵，再写业务；能力缺失的降级方案提前设计。
- 心智迁移的验收标准：能在 10 分钟内给 Web 同事讲清"为什么我这里没有 router 和 axios"。

## 相关笔记

- 上级：[[小程序开发/uniapp应用开发/uniapp应用开发|uniapp应用开发]]
- [[小程序开发/微信小程序开发/双线程架构与项目结构|双线程架构与项目结构（原理侧）]]

## 阅读导航

**推荐阅读顺序：第 2 / 13 篇**（完整顺序见 [[小程序开发/小程序开发|小程序开发 MOC]]）

- ⬅️ 上一篇：[[小程序开发/微信小程序开发/双线程架构与项目结构|双线程架构与项目结构]]
- ➡️ 下一篇：[[小程序开发/微信小程序开发/视图层-WXML-WXSS与常用组件|视图层：WXML/WXSS 与常用组件]]
