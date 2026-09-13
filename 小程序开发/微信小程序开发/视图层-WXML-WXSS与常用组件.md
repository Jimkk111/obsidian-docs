---
title: 视图层：WXML/WXSS 与常用组件
created: 2026-09-13
updated: 2026-09-13
source: 微信官方文档 developers.weixin.qq.com/miniprogram/dev/framework/view/
tags:
  - 小程序开发
  - 微信小程序
---

# 视图层：WXML/WXSS 与常用组件

## WXML 语法

WXML 不是 HTML：**只有小程序定义的标签**（view/text/image/...），语法向 Vue 靠拢但属于自己的一套（`wx:` 指令）。

```html
<!-- 数据绑定 -->
<text>{{ message }}</text>

<!-- 列表渲染：wx:key 必给（字符串或保留字 *this），否则告警且 diff 低效 -->
<view wx:for="{{ dishList }}" wx:key="id">
  {{ index }} - {{ item.name }}
</view>
<!-- block 是无渲染包裹标签，常与 wx:for / wx:if 组合 -->
<block wx:for="{{ list }}" wx:key="id">{{ item.name }}</block>

<!-- 条件渲染 -->
<view wx:if="{{ score >= 60 }}">及格</view>
<view wx:elif="{{ score >= 40 }}">补考</view>
<view wx:else>重修</view>
<!-- hidden 是切 display，wx:if 是真卸载：频繁切换用 hidden，条件少变用 wx:if -->
<view hidden="{{ !visible }}">常驻元素</view>
```

模板与引用：

- `<template name="t">` 定义，`<template is="t" data="{{...item}}"/>` 使用（纯展示复用，无逻辑）。
- `import` 引入目标文件的 template（不含嵌套）；`include` 拷贝整个标签结构（除 template/wxs）。
- `<wxs>` 模块：在视图层运行的 JS 子集，用于视图侧的数据格式化（弥补 WXML 表达式能力弱，且运行在渲染层无通信开销）。

## WXSS 与 rpx

- **rpx（responsive pixel）**：规定屏宽 = 750rpx。在 iPhone6（375px）下 `1rpx = 0.5px`。设计稿 750px 宽可直接 1:1 写 rpx——这是小程序版的"rem 自动化"。
- 仅支持部分选择器：类、id、element、`::before/::after`；**不支持通配符 `*`、属性选择器等**。
- `@import "xxx.wxss";` 引入样式；页面样式默认只作用本页，全局样式写 `app.wxss`（页面样式优先级更高）。
- 样式隔离：自定义组件默认样式隔离，可通过 `styleIsolation` 调整（`isolated / apply-shared / shared`）。
- 布局以 flex 为主流（官方也建议），float 可用但没必要。
- 安全区适配：`padding-bottom: env(safe-area-inset-bottom);`（iPhone 底部小黑条）。

## 常用内置组件

### 视图容器

- `view`：块状容器，等价 div。
- `scroll-view`：**局部滚动必用**（页面级滚动不需要）。核心属性：
  - `scroll-y`/`scroll-x` 方向；`upper-threshold`/`lower-threshold` + `bindscrolltoupper/bindscrolltolower` 做触顶刷新/触底加载；
  - `scroll-into-view="id值"`：滚动到指定子元素（**左右联动菜单的实现核心**）；
  - `scroll-top`：数值控制滚动位置。
- `swiper` + `swiper-item`：轮播图，`autoplay/circular/indicator-dots`。
- `movable-area`/`movable-view`：可拖拽区域。

### 基础内容

- `text`：**唯一可以长按选中、可嵌套 text 的文本组件**；`user-select` 开启复制；`space` 控制空格显示。纯文本放 text 而不是 view 里是规范写法。
- `image`：默认 320×240，**必须自己给宽高或用 mode**。常用 `mode="aspectFill"`（裁剪填满）/`aspectFit`（完整显示）/`widthFix`（宽定高自适应——列表图最常用）；`lazy-load` 懒加载（仅 page 内）；支持 webp。
- `rich-text`：渲染 HTML 片段（受控节点白名单）。

### 表单

- `input`/`textarea`：小程序的输入是非受控观感（值由原生组件维护），用 `value` + `bindinput` 同步；`type="nickname"` 是官方昵称填写能力的一部分。
- `picker`：底部弹起选择器（mode: selector/time/date/region），`bindchange` 拿值。
- `button`：`open-type` 直通微信能力——`contact` 客服、`getPhoneNumber` 手机号（企业认证）、`chooseAvatar` 头像、`share` 转发、`getUserInfo`（已废弃）。`form` + `report-submit` 做表单 id 收集。

### 媒体与地图

- `video`/`camera`/`live-player`：同层渲染（原生组件不再插入在 WebView 最顶层，普通 z-index 可盖住）。
- `map`：地图组件，markers/polyline，配 `wx.createMapContext` 操作。

## 自定义导航栏（沉浸式）

`app.json` 配 `"navigationStyle": "custom"` 后系统导航栏消失，需自己实现：

1. 状态栏高度：`wx.getWindowInfo().statusBarHeight`（旧 API 为 getSystemInfoSync）。
2. 胶囊位置：`wx.getMenuButtonBoundingClientRect()` 拿胶囊 top/height，导航内容区高度常取 `胶囊top - 状态栏高 + 胶囊height + 底部间距`。
3. 全屏背景延伸：直接把背景画到顶部，内容区用 padding 撑开。

这是小程序端特有的布局知识，Web 端没有对应物。

## 经验提示

- `image` 不给宽高是新手第一大坑（不显示或尺寸错乱）。
- 页面级下拉刷新要在页面 json 里 `enablePullDownRefresh: true`，然后 `onPullDownRefresh` 里处理完调 `wx.stopPullDownRefresh()`。
- 触底加载三件套：`onReachBottom` + 页码 state + 加载锁（防止重复请求）。

## 相关笔记

- 上级：[[小程序开发/微信小程序开发/微信小程序开发|微信小程序开发]]
- [[小程序开发/微信小程序开发/逻辑层-Page-Component与生命周期|逻辑层：Page/Component 与生命周期]]

## 阅读导航

**推荐阅读顺序：第 3 / 13 篇**（完整顺序见 [[小程序开发/小程序开发|小程序开发 MOC]]）

- ⬅️ 上一篇：[[小程序开发/uniapp应用开发/Web开发差异对照表|Web 开发差异对照表]]
- ➡️ 下一篇：[[小程序开发/微信小程序开发/逻辑层-Page-Component与生命周期|逻辑层：Page/Component 与生命周期]]
