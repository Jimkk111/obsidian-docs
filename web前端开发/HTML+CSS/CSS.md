---
title: CSS
source: https://www.yuque.com/mook-mvqcp/wc9wsu/lng0rgnbdbfm366v
created: 2026-05-25
updated: 2026-09-07
tags:
  - HTML+CSS
---

> [!info] 目录说明
> 目录结构参照 MDN Web Docs 的 [CSS 学习区](https://developer.mozilla.org/zh-CN/docs/Learn_web_development/Core/Styling_basics) 与 [CSS 参考](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference)，并参考《CSS 权威指南（第 4 版）》补齐面试高频主题。标注「待补充」的小节为尚未填充内容，按需逐步完善。

## 一、CSS 基础

### 1. 引入方式（待补充）

行内样式、内嵌样式表、外部样式表、`@import` 与 `<link>` 的区别。

### 2. 语法与规则集（待补充）

选择器 + 声明块、注释、@规则（@media、@font-face 等）。

### 3. 选择器与优先级（待补充）

基础选择器、组合器、属性选择器、伪类与伪元素；特异性计算、`!important`、层叠与继承。

### 4. 值与单位（待补充）

长度单位（px、em、rem、vw / vh、%）、颜色值 → 详见 [[web前端开发/HTML+CSS/颜色|颜色]]。

## 二、盒模型

### 1. content、padding、border、margin（待补充）

### 2. box-sizing 与元素宽高（待补充）

content-box 与 border-box 的区别。

### 3. margin 合并（外边距塌陷）（待补充）

发生场景与解决方法。

### 4. BFC 块级格式化上下文（待补充）

触发条件与应用场景。

## 三、文本与字体

### 1. 字体属性 font（待补充）

font-family、font-size、font-weight、@font-face 与字体加载。

### 2. 文本属性 text（待补充）

color、text-align、line-height、text-decoration、text-overflow 与单行/多行省略。

## 四、背景、边框与视觉效果

### 1. background 系列（待补充）

background-color / image / position / size / repeat / attachment、多背景、渐变。

### 2. 边框与圆角（待补充）

border、border-radius、border-image、outline 与 border 的区别。

### 3. 阴影（待补充）

box-shadow、text-shadow。

### 4. overflow 与裁剪（待补充）

overflow 的取值、文本溢出处理。

### 5. filter 滤镜

filter 属性是滤镜属性，其取值 `blur()` 表示模糊半径，可以搭配 inset 使用（作用是把元素拉大到大于背景，因为 blur() 需要参考附近像素计算模糊，如果大小和背景一样大，靠边界的区域就没有模糊效果），打造盖在背景上的模糊层。

## 五、定位与层叠

### 1. position 定位（待补充）

static、relative、absolute、fixed、sticky 的定位基准与使用场景。

### 2. inset 属性

inset 属性很实用，是 top、bottom、right 和 left 的简写，可以控制定位元素的大小。

### 3. z-index 与层叠上下文（待补充）

## 六、Flex 布局（待补充）

### 1. 容器属性（待补充）

display:flex、flex-direction、flex-wrap、justify-content、align-items、align-content、gap。

### 2. 项目属性（待补充）

flex-grow、flex-shrink、flex-basis、flex 简写、align-self、order。

### 3. 常见布局场景（待补充）

水平垂直居中、圣杯布局、粘性页脚等。

## 七、Grid 布局

### 1. 基础概念

- 网格线：划分行列的分界线。
- 网格轨道：两条网格线围成的区域。
- 网格单元：一条列轨道和一条行轨道交叉形成的区域。
- 网格区域：任意四条网格线围成的封闭区域。
- 网格容器：开启 `display: grid` 的元素。
- 网格项：网格容器的直接子元素。

### 2. 样式和函数、关键字

- `display: grid`：开启网格布局。
- `grid-template-rows`、`grid-template-columns`：定义网格行和网格列。
- `grid-auto-rows`、`grid-auto-columns`：隐式行、隐式列。
- `repeat()`：用于声明多个尺寸相同的行或列。
- `auto-fill` 和 `auto-fit`：当不确定一行或者一列的数量时，用做 repeat() 的第一个参数。
- `minmax()`：声明一个数值范围，第一个参数是最小值，第二个参数是最大值。
- `fr`：剩余空间的比例单位。
- `grid-row`、`grid-column`：用在网格项目，声明跨行、跨列。
- `grid-auto-flow`：决定隐式网格的排列方向以及是否启用密集填充（`dense`）。
- `gap`、`row-gap`、`column-gap`。

### 3. 对齐

justify 表示水平方向，align 表示垂直方向。place 是简写，先垂直后水平。

整个网格在容器内的对齐：justify-content、align-content，写在容器的选择器。

单个项目在单元格内的对齐：justify-items、align-items，写在容器的选择器，控制所有单元格；justify-self、align-self，写在项目的选择器，只作用这一个单元格。

### 4. 应用

grid 布局实现不了瀑布流，瀑布流用多列布局实现。

## 八、其他布局方式

### 1. 普通流与文档流（待补充）

### 2. 浮动 float（待补充）

浮动的作用、清除浮动的方式。

### 3. 多列布局

- `column-count`：声明列数。
- `column-width`：声明列宽。
- `column-gap`：列间隙。
- `column-rule`：列分隔线，类似 border 属性。

## 九、响应式布局

### 1. 媒体查询 @media（待补充）

### 2. viewport 与移动端适配（待补充）

rem 适配方案、vw / vh 方案、 flexible / postcss-pxtorem。

### 3. 响应式单位与函数（待补充）

%、em / rem、vw / vh、`calc()`、`min()` / `max()` / `clamp()`。

## 十、过渡、变换与动画

### 1. transition 过渡（待补充）

属性、时长、缓动函数、延迟；与动画的触发条件。

### 2. transform 变换（待补充）

translate、rotate、scale、skew、3D 变换；transform 不引起重排。

### 3. animation 动画（待补充）

@keyframes、animation 属性、与 transition 的区别。

## 十一、CSS 变量与进阶特性

### 1. 自定义属性 CSS 变量（待补充）

`--var` 定义、`var()` 使用、作用域。

### 2. CSS3 新特性汇总（待补充）

### 3. 伪类与伪元素（待补充）

常用伪类（:hover、:nth-child、:not、:focus-within 等）、::before / ::after 的应用。

## 十二、CSS 常见面试问题

### 1. 水平垂直居中的实现方式（待补充）

### 2. 重排（回流）与重绘（待补充）

触发时机、如何减少。

### 3. 隐藏元素的方式与区别（待补充）

display: none、visibility: hidden、opacity: 0。

### 4. Flex 与 Grid 的区别与选型（待补充）

### 5. CSS 选择器优先级如何计算（见 [[#一、CSS 基础]]）

---

上级：[[web前端开发/HTML+CSS/HTML+CSS|HTML+CSS]]

## 笔记

- [[web前端开发/HTML+CSS/八股文问题|八股文问题]]
- [[web前端开发/HTML+CSS/颜色|颜色]]
