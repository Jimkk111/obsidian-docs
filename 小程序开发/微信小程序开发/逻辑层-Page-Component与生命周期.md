---
title: 逻辑层：Page/Component 与生命周期
created: 2026-09-13
updated: 2026-09-13
source: 微信官方文档 developers.weixin.qq.com/miniprogram/dev/reference/api/Page.html
tags:
  - 小程序开发
  - 微信小程序
---

# 逻辑层：Page/Component 与生命周期

## Page：页面构造器

```js
Page({
  data: { count: 0 },

  onLoad(options) {
    // 页面加载，仅一次；options = 路由跳转携带的 query 参数
    // 页面级初始化：读参数、发首屏请求
  },
  onShow() {
    // 每次进入页面都触发（含从其他页返回、切后台回来）
    // 适合做"返回刷新"：如从地址页返回订单页回显新地址
  },
  onReady() {
    // 首次渲染完成，仅一次；可安全获取节点尺寸（SelectorQuery）
  },
  onHide() {},   // 页面隐藏（navigateTo 跳走 / 切后台）
  onUnload() {}, // 页面卸载（redirectTo/navigateBack）；清理定时器、解绑监听

  onPullDownRefresh() {
    // 需页面 json 或 app.json 配置 enablePullDownRefresh: true
    // 处理完必须调 wx.stopPullDownRefresh()
  },
  onReachBottom() {},      // 触底，距离由 onReachBottomDistance 配置
  onPageScroll() {},       // 页面滚动，高频回调，别在里面做复杂计算
  onShareAppMessage() {},  // 定义后右上角菜单才有"转发"
  onShareTimeline() {},    // 朋友圈分享
})
```

### setData：唯一的数据更新通道

直接 `this.data.count = 1` 不会更新视图（且会破坏一致性）。必须：

```js
this.setData({ count: 1 })
// 支持数据路径写法，精准更新数组某一项（大列表优化的关键）
this.setData({ 'list[2].name': '新名字', 'obj.a.b': 3 })
```

**setData 性能规则**（官方明确强调）：

1. 不要频繁调用（超过 20ms 一次的调用会阻塞渲染），合并批量更新。
2. 不要一次传大量新数据（传输有序列化成本），只传变化的字段。
3. 不要把**后台不参与渲染的数据**放进 data/setData——用纯数据字段：

```js
Component({
  options: { pureDataPattern: /^_/ },
  data: { _raw: '不进视图层', shown: '' },
})
```

## Component：自定义组件构造器

```js
Component({
  properties: {
    title: { type: String, value: '' },       // 父传子
  },
  data: {},
  methods: {
    onTap() {
      this.triggerEvent('customevent', { id: 1 })  // 子传父：自定义事件
    },
    getChild() {
      this.selectComponent('#child')               // 父直接拿子组件实例（慎用）
    },
  },
  lifetimes: {
    created() {},   // 实例创建，还不能 setData
    attached() {},  // 进入页面节点树，等价"挂载前"
    ready() {},     // 布局完成，可获取节点
    detached() {},  // 离开节点树，清理资源
  },
  pageLifetimes: {
    show() {},      // 组件所在页面 onShow 时
    hide() {},
  },
})
```

- 组件的 WXML 里 `bindtap="onTap"` 直接绑定 methods 中方法（页面的事件处理函数也写在 Page 对象上，不是 methods）。
- `behaviors`：类似 Vue mixin，实现代码复用（含生命周期钩子合并）。
- `slot` 支持单个/具名插槽（需 `options: { multipleSlots: true }`）。
- 组件间跨层通信可用 `relations`（关联组件）或 `selectComponent`，但优先用 props/事件的数据流。

## 生命周期全景图

三套生命周期叠加，答题/排查都要分清归属：

```
应用级（app.js App()）：onLaunch → onShow →（onHide ⇄ onShow）…（仅一次 onLaunch）
页面级（Page）：onLoad → onShow → onReady →（onHide ⇄ onShow）→ onUnload
组件级（Component）：created → attached → ready →（moved/detached）
```

典型时序题：**A 页 navigateTo 到 B 页**——`A.onHide → B.onLoad → B.onShow → B.onReady`；**B navigateBack 回 A**——`B.onUnload → A.onShow`。

## 经验提示

- 需要跨页刷新时优先想 `onShow` + 标记位，而不是页面栈强改数据。
- `onUnload` 里清定时器/取消订阅（WebSocket、事件总线），否则页面栈外的"幽灵回调"会引发诡异 bug。
- 组件里**没有** onLoad/onShow（那是页面的），组件用 lifetimes；组件想感知页面显隐用 pageLifetimes。

## 相关笔记

- 上级：[[小程序开发/微信小程序开发/微信小程序开发|微信小程序开发]]
- [[小程序开发/微信小程序开发/路由与页面栈|路由与页面栈]]

## 阅读导航

**推荐阅读顺序：第 4 / 13 篇**（完整顺序见 [[小程序开发/小程序开发|小程序开发 MOC]]）

- ⬅️ 上一篇：[[小程序开发/微信小程序开发/视图层-WXML-WXSS与常用组件|视图层：WXML/WXSS 与常用组件]]
- ➡️ 下一篇：[[小程序开发/微信小程序开发/路由与页面栈|路由与页面栈]]
