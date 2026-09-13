---
title: API 体系与条件编译
created: 2026-09-13
updated: 2026-09-13
source: uni-app 官方文档 uniapp.dcloud.net.cn/api/
tags:
  - 小程序开发
  - uniapp
---

# API 体系与条件编译

## uni.* API 体系

所有平台能力统一前缀 `uni.`，语义对齐小程序 API，编译到其他端由运行时映射实现：

| 分类 | 代表 API | Web 端对应 |
|---|---|---|
| 网络 | `uni.request` / `uploadFile` / `downloadFile` / `connectSocket` | XHR / fetch |
| 存储 | `uni.setStorageSync` / `getStorageSync` / `removeStorageSync` | localStorage |
| 交互 | `showToast` / `showModal` / `showActionSheet` / `showLoading` | alert/confirm 无原生对应 |
| 导航 | `navigateTo` / `redirectTo` / `switchTab` / `navigateBack` / `reLaunch` | router.push |
| 设备 | `getWindowInfo` / `getNetworkStatusChange` / `onNetworkStatusChange` | window/screen/navigator |
| 登录支付 | `login` / `getUserProfile`(废弃) / `requestPayment` | 无（平台能力） |

特征与注意：

- **大多为回调风格**，官方提供 `uni.xxx` 的 Promise 化（部分 API 需 `callback` 与 `success/fail` 二选一）；自己封装统一 Promise 层是项目标配：

```js
export function request({ url = '', params = {}, method = 'GET' }) {
  return new Promise((resolve, reject) => {
    uni.request({
      url: baseUrl + url,
      data: params,
      method,
      header: { 'content-type': 'application/json', Cookie: 'JSESSIONID=' + userStore.sessionId },
      success: (res) => {
        if (res.statusCode === 401) return handleSessionExpired()
        res.data.code === 1 || res.data.code === 200 ? resolve(res.data) : reject(res.data)
      },
      fail: (err) => reject({ code: -1, msg: '网络异常', data: err }),
    })
  })
}
```

- **axios 不可直接用**（依赖 XHR/DOM）：H5 端勉强可跑但小程序端必挂，统一用 uni.request 封装。
- 域名白名单约束与原生小程序一致（小程序端 HTTPS + 备案 + 后台配置）。
- `uni.connectSocket` 只有一个默认连接（同域覆盖），多连接/STOMP 场景需自己管理 socket 实例或给 stomp.js 传适配器。

## 条件编译：跨端差异的官方解法

**`#ifdef` / `#ifndef` / `#endif` 注释块**，在 js、模板、样式、json、manifest 中均可用：

```js
// #ifdef MP-WEIXIN
wx.showToast({ title: '仅微信端执行' })   // 也可写 uni.
// #endif

// #ifndef H5
console.log('非 H5 端才编译进来')
// #endif
```

```html
<view>
  <!-- #ifdef H5 -->
  <button @click="browserShare">网页分享</button>
  <!-- #endif -->
</view>
```

常用平台标识：`MP-WEIXIN`、`MP-ALIPAY`、`H5`、`APP-PLUS`（App）、`APP-PLUS-NVUE`。多端组合：`#ifdef H5 || MP-WEIXIN`。

运行时判断（值判断而非编译剔除，尽量少用）：

```js
const platform = process.env.UNI_PLATFORM   // 'mp-weixin' | 'h5' | ...
if (import.meta.env.UNI_PLATFORM === 'h5') { /* Vue3/Vite 下 */ }
```

**原则：能用条件编译静态剔除，就不用运行时 if**——编译掉的代码不进产物，运行时判断两端代码都保留。

## 平台差异的常见清单

- 生命周期：`onBackPress` 微信端不支持；分享钩子仅小程序/对应端生效。
- API 能力：支付、登录、订阅消息只在小程序/App 端有意义，H5 端调用直接失败。
- 组件表现：`input` 各端键盘行为不同；导航栏、tabBar 只有小程序/App 是原生的，H5 端由框架模拟渲染。
- 样式：rpx 全端通用；`position: fixed` 在小程序页面滚动上下文中行为有细节差异（相对于视口而非页面内容）。

## 状态管理（Vue3 → Pinia）

```js
// stores/user.js
import { defineStore } from 'pinia'
export const useUserStore = defineStore('user', {
  state: () => ({
    sessionId: uni.getStorageSync('sessionId') || '',   // 持久化恢复
  }),
  actions: {
    setSession(id) {
      this.sessionId = id
      uni.setStorageSync('sessionId', id)   // 无官方 persist 插件跨端通用，手工封装
    },
    logout() {
      this.sessionId = ''
      uni.removeStorageSync('sessionId')
    },
  },
})
```

- Vuex（Vue2 项目）→ Pinia（Vue3 项目）迁移动机与 Web 端一致：TS 友好、无 mutation 样板、组合式 API 原生融合。
- main.js 必须 `return { app, Pinia }`（uni-app 约定，漏返回 Pinia 在小程序端会报 Pinia 未安装）。
- 持久化没有跨端通用的官方插件，惯例是 action 内手工同步 storage（见上）。

## 经验提示

- 封装请求层时把"登录失效"（401/自定义失效码）收敛到一处：清 Pinia 登录态 + reLaunch 首页，业务代码只管成功路径。
- 断网处理：App.vue `onLaunch` 里 `uni.onNetworkStatusChange` 全局监听，断网跳独立提示页（或用 showToast），恢复后返回。

## 相关笔记

- 上级：[[小程序开发/uniapp应用开发/uniapp应用开发|uniapp应用开发]]
- [[小程序开发/uniapp应用开发/微信登录链路与支付|微信登录链路与支付]]

## 阅读导航

**推荐阅读顺序：第 11 / 13 篇**（完整顺序见 [[小程序开发/小程序开发|小程序开发 MOC]]）

- ⬅️ 上一篇：[[小程序开发/uniapp应用开发/组件-easycom与样式适配|组件：easycom 与样式适配]]
- ➡️ 下一篇：[[小程序开发/uniapp应用开发/微信登录链路与支付|微信登录链路与支付]]
