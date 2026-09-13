---
title: 组件：easycom 与样式适配
created: 2026-09-13
updated: 2026-09-13
source: uni-app 官方文档 uniapp.dcloud.net.cn/component/；uni-ui 文档
tags:
  - 小程序开发
  - uniapp
---

# 组件：easycom 与样式适配

## 组件规范

uni-app 的内置组件**靠近小程序规范**而非 HTML：模板里写 `<view>/<text>/<image>/<scroll-view>/<swiper>/<input>/<button>`，编译到 H5 时才映射为 div/span/img。写模板时按"无 DOM"的心智，任何端都安全。

| Web 习惯 | uni-app 写法 | 备注 |
|---|---|---|
| div | view | 块容器 |
| span / 纯文本 | text | 文本必须包在 text 中（选中/复制能力也靠它） |
| img | image | 默认 320×240，必须定宽高或 `mode="widthFix"` |
| 滚动容器 overflow:auto | scroll-view | 配 scroll-y/scroll-x + bindscrolltolower |
| ul/li 列表 | view + wx:for / v-for | wx:key 或 :key 必给 |
| a 链接 | navigator 组件 / uni.navigateTo | 无浏览器跳转概念 |

Vue 指令照常：`v-if/v-for/v-model/v-bind/v-on` 全部可用（编译期转成各端对应物）。

## easycom：约定优于配置的自动注册

```json
// pages.json
"easycom": {
  "autoscan": true,
  "custom": {
    "^uni-(.*)": "@/components/uni-$1/uni-$1.vue"
  }
}
```

- `autoscan: true`：`components/组件名/组件名.vue` 目录结构的组件**免 import 免注册**，模板直接使用。
- `custom` 正则：把 `uni-xxx` 映射到指定路径——引入 uni-ui 等 uni_modules 组件库时靠它自动解析。
- 收益：去掉满屏的 `import xxx from '@/components/xxx.vue'` + `components: {}`；组件用不到时**不会打包**（按需打包天然成立）。
- 注意：easycom 是编译期静态解析，组件名必须静态写死（动态 `<component :is>` 场景仍需手动 import）。

## 组件来源三层

1. **内置组件**：编译器自带，各端原生对应物（view/scroll-view/picker/...）。
2. **uni_modules 扩展组件**（uni-ui）：`uni-icons`、`uni-nav-bar`、`uni-datetime-picker`、`uni-popup` 等，插件市场导入 uni_modules 目录，easycom 自动注册——比手拷贝旧版组件文件规范得多（旧项目把组件文件整个拷进 components 是历史反模式）。
3. **自定义组件**：普通 Vue SFC，放 `components/` 下遵守 easycom 结构即可。

## 样式体系

### rpx 与适配

- 规则与小程序一致：**750rpx = 屏幕宽度**。设计稿 750px 宽时数值 1:1 抄成 rpx；uni-app 编译器在 H5 端也会把 rpx 换算为 vw/px，跨端一致。
- 字号建议：正文用 rpx 或 px 均可，但**边框 1px 慎用 1rpx**（部分机型渲染过细丢失）。

### 全局样式与变量

- `App.vue` 的 `<style>`（不加 scoped）= 全局样式，等价 app.wxss。
- `uni.scss`：全局 SCSS 变量文件，**编译期自动注入每个组件**，模板/样式中直接用变量名，无需 import。官方预置一批色板变量，业务主色（如 `#ffc200`）应收敛到这里，替代各页面散落硬编码。
- 页面 `<style lang="scss">` 使用嵌套、变量、mixin（dart-sass 编译）。

### 样式穿透与隔离

- 小程序端组件默认样式隔离；要覆盖组件库内部样式用 `:deep()`（Vue3 编译器会转为正确写法）：

```scss
.parent :deep(.uni-navbar__content) { background: #333; }
```

- 历史写法 `/deep/` 与 `::v-deep` 在 dart-sass/Vue3 下会报错或弃用——迁移老项目时是必改项（本项目摸底就改了 3 处）。

### 状态栏/安全区适配

- CSS 变量：`--status-bar-height`（状态栏高）、`--window-top`（标题栏高）、`--window-bottom`。
- 底部安全区：`padding-bottom: env(safe-area-inset-bottom);`。
- 自定义导航栏（`navigationStyle: "custom"`）需要自己用 `uni.getWindowInfo().statusBarHeight` + `uni.getMenuButtonBoundingClientRect()`（微信端）计算内容区，或直接用 uni-nav-bar 组件。

### 条件编译样式

```scss
/* #ifdef H5 */
.web-only { display: none; }
/* #endif */
```

样式、模板、JS、甚至 pages.json 字段都可以条件编译（见 API 体系与条件编译笔记）。

## 经验提示

- 布局用 flex 全覆盖，别依赖 float/表格布局（跨端渲染差异最小的就是 flex）。
- `image` 组件永不给宽高 = 不显示，是 uni-app/小程序第一大新手坑。
- 组件库优先走 uni_modules + easycom，手拷贝源码的"离线组件"会失去升级能力且容易带着旧写法（Vue2 语法）在 Vue3 编译时报错。

## 相关笔记

- 上级：[[小程序开发/uniapp应用开发/uniapp应用开发|uniapp应用开发]]
- [[小程序开发/uniapp应用开发/API体系与条件编译|API 体系与条件编译]]

## 阅读导航

**推荐阅读顺序：第 10 / 13 篇**（完整顺序见 [[小程序开发/小程序开发|小程序开发 MOC]]）。easycom 与条件编译是 uni-app 框架层真正的新东西，本篇与下一篇建议放慢精读。

- ⬅️ 上一篇：[[小程序开发/uniapp应用开发/路由与页面栈|路由与页面栈]]
- ➡️ 下一篇：[[小程序开发/uniapp应用开发/API体系与条件编译|API 体系与条件编译]]
