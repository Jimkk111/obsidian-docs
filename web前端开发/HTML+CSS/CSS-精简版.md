---
title: CSS 精简版
created: 2026-09-08
updated: 2026-09-08
tags:
  - HTML+CSS
---

> [!info]
> 快速复习版，只保留结论和考点。详细讲解见 [[web前端开发/HTML+CSS/CSS|CSS]]。

## 一、CSS 基础

### 1. 引入方式

行内 style 属性（优先级高、不复用）、`<style>` 内嵌、`<link>` 外部（推荐，可缓存）。`@import` 与 `<link>` 的区别：link 是 HTML 标签**并行**加载，@import 是 CSS 语法、解析到才**串行**请求（易 FOUC）；@import 优先级低于其他规则、必须写在文件最前。

### 2. 语法与规则集

规则集 = 选择器 + `{ 属性: 值; }`；注释只有 `/* */`。@规则：@media、@import、@font-face、@keyframes、@supports、@page。

### 3. 选择器与优先级

**基础**：`*` 通配、`p` 标签、`.box` 类、`#id`、属性选择器（`[attr]`、`[attr="v"]`、`[attr~="v"]` 含词、`[attr|="v"]` 等于或 v- 开头、`^=` 开头、`$=` 结尾、`*=` 包含）。

**组合器**：空格后代、`>` 直接子代、`+` 紧邻兄弟、`~` 后续所有兄弟。

**伪类**（单冒号，选中某状态的已有元素）：交互 `:hover :focus :active :focus-within`；UI `:checked :disabled :required :valid :invalid :placeholder-shown`；结构 `:root :empty :first-child :last-child :nth-child(an+b)`（可 odd/even）`:nth-last-child` `:nth-of-type`（只数同类型标签，与 nth-child 的区别）；逻辑 `:not()` `:is()` `:where()`（同 is 但优先级为 0）`:has()`（父选择器）。

**伪元素**（双冒号，制造不存在的元素）：`::before` / `::after`（必须有 content，默认行内）、`::first-line`、`::first-letter`、`::selection`、`::placeholder`、`::marker`。

**优先级**四位比较：行内 1-0-0-0 > id > 类/伪类/属性 > 元素/伪元素；通配、组合器、`:where()` 不计位。同位看**源码顺序**（所以伪类按 LVHA 顺序写）。`!important` > 一切普通规则。**继承值没有优先级**，会被任何直接命中的规则覆盖。默认继承：color、font 系列、line-height、text-align、visibility；盒模型属性不继承。

### 4. 值与单位

px 是 CSS 像素不是物理像素。em：font-size 上相对**父元素**，其他属性相对**自身** font-size。rem 相对根元素。vw/vh 是视口宽/高的 1%。% 参照物因属性而异（margin/padding 垂直方向也相对**宽度**）。颜色 → 详见 [[web前端开发/HTML+CSS/颜色|颜色]]。

## 二、盒模型

### 1. content、padding、border、margin

由内到外四层：content、padding（背景延伸到此）、border、margin（透明间隔）。padding 不可负、margin 可负；`margin: 0 auto` 只能水平居中。

### 2. box-sizing 与元素宽高

content-box（默认）：width 只算内容区，实际宽 = width + padding + border。border-box：width 含 padding + border，更符合直觉，全局 `* { box-sizing: border-box; }`。

### 3. margin 合并（外边距塌陷）

垂直方向、普通流的块级盒之间：相邻兄弟取大者；**父子之间**（子元素 margin 穿透父元素）；空块自身。解决：父元素开启 BFC（推荐 `display: flow-root`）、父子间加 border/padding 隔断、改用 flex/grid（不合并）。

### 4. BFC 块级格式化上下文

独立渲染区域，内部布局不影响外部。触发：根元素、float、absolute/fixed、**display: flow-root**（无副作用，推荐）、inline-block、flex/grid、overflow 非 visible。应用：**包含内部浮动（解决高度塌陷）**、不与浮动重叠（自适应两栏）、隔离 margin 合并。

## 三、文本与字体

### 1. 字体属性 font

font-family 写字体栈，末尾放通用族（sans-serif 等），中英文混排英文字体在前。font-weight 100~900。`font` 简写 **size 和 family 必填**。`@font-face` 自定义字体：woff2 优先，`font-display: swap` 先显示回退字体避免文字不可见。

### 2. 文本属性 text

line-height 推荐无单位数字（按各自字号继承）。text-align 作用于块级元素控制内部行内内容。**单行省略**：`overflow: hidden; white-space: nowrap; text-overflow: ellipsis;`。**多行省略**：`display: -webkit-box; -webkit-box-orient: vertical; -webkit-line-clamp: n; overflow: hidden;`。

## 四、背景、边框与视觉效果

### 1. background 系列

多背景**先写的在上层**；渐变属于 image 类型。position 百分比 = 图片的 x% 点对准容器的 x% 点（50% 即居中）。size：**cover 完全覆盖可能裁剪，contain 完整显示可能留白**。clip: text 文字镂空。简写中 position/size 用斜杠分隔：`background: url() center/cover no-repeat`。

### 2. 边框与圆角

border **style 必填**（默认 none）；宽高为 0 + 三边 border 可画三角形。radius 四值顺时针（左上起），50% 变圆/椭圆。**outline 与 border**：outline 不占空间、不能单边设置、用于 :focus 轮廓。

### 3. 阴影

`box-shadow: x y 模糊 扩散 颜色 inset;`，多层逗号分隔，不占布局空间。text-shadow 无扩散、无 inset。

### 4. overflow 与裁剪

visible / hidden / scroll（恒显滚动条）/ auto（按需）/ clip。一轴 hidden 时另一轴 visible 变 auto。overflow 非 visible 会**创建 BFC**。hidden 会裁掉溢出的定位子元素；父级 overflow 非 visible 会让 **sticky 失效**。

### 5. filter 滤镜

filter 属性是滤镜属性，其取值 `blur()` 表示模糊半径，可以搭配 inset 使用（作用是把元素拉大到大于背景，因为 blur() 需要参考附近像素计算模糊，如果大小和背景一样大，靠边界的区域就没有模糊效果），打造盖在背景上的模糊层。

## 五、定位与层叠

### 1. position 定位

| 取值 | 定位基准 | 脱流 |
|---|---|---|
| static | 无（offset 无效） | 否 |
| relative | 自身原位置（空间保留） | 否，常作 absolute 的基准 |
| absolute | 最近非 static 祖先，否则初始包含块 | 是 |
| fixed | 视口 | 是；**祖先有 transform/filter 时失效**（改相对该祖先） |
| sticky | 滚动阈值前 relative、之后 fixed | 否；需写 top 等阈值，父级 overflow 非 visible 失效 |

### 2. inset 属性

inset 属性很实用，是 top、bottom、right 和 left 的简写，可以控制定位元素的大小。

### 3. z-index 与层叠上下文

只对 **position 非 static 及 flex/grid 项**生效。层叠上下文是独立"图层"：内部 z-index 只在**本上下文内**比较，整个上下文作为整体参与外部层叠（父级 z-index 低，子级再大也盖不过外面）。创建条件：根元素；z-index 非 auto 的定位元素；**opacity < 1**；transform、filter、will-change、isolation: isolate 等。排查 z-index 失效：先看是否非 static，再沿祖先找多余的层叠上下文。

## 六、Flex 布局

### 1. 容器属性

`display: flex` 后子元素 float/clear/vertical-align 失效、margin 不合并。**主轴**由 flex-direction 决定（row/column 及 reverse），与之垂直的是**交叉轴**。flex-wrap：nowrap 默认压缩不换行。justify-content 管主轴：center、space-between（两端）、space-around（两侧间距是中间一半）、space-evenly（完全相等）。align-items 管交叉轴：**stretch 默认拉伸（子项默认等高的原因）**、center、baseline。align-content 只在多行时生效。gap 设项目间距。

### 2. 项目属性

flex-grow 剩余空间放大（默认 0）、flex-shrink 不足时缩小（默认 1）、flex-basis 基准尺寸（优先于 width）。简写必背：**`flex: 1` = 1 1 0%（均分/自适应栏）**、`flex: auto` = 1 1 auto、`flex: none` = 0 0 auto（固定不缩放）。align-self 覆盖单个项目的交叉轴对齐；order 改视觉顺序（可负，小者在前）。

### 3. 常见布局场景

水平垂直居中：`justify-content: center + align-items: center`。左固定右自适应：左固定宽 + `flex: none`，右 `flex: 1`。粘性页脚：容器 `min-height: 100vh + column`，内容区 `flex: 1`。某元素吸底靠右：`margin-left: auto`（auto margin 吞掉剩余空间）。

## 七、Grid 布局

### 1. 基础概念

网格线（编号、是项目定位坐标）→ 两条线围成轨道（行/列）→ 行列交叉成网格单元 → 四条线围成网格区域。`display: grid` 的是**网格容器**，**直接子元素**才是网格项。

### 2. 样式和函数、关键字

`grid-template-rows/columns` 定义显式行列，`grid-auto-rows/columns` 定义超出部分的隐式行列。`fr` 是剩余空间比例单位；`repeat(3, 1fr)` 重复声明；**`repeat(auto-fill, minmax(200px, 1fr))` 是响应式卡片网格黄金搭配**；`minmax(min, max)` 尺寸范围。项目用 `grid-row/column` 跨轨道（`1 / 3` 或 `span 2`）。`grid-auto-flow` 排列方向，加 `dense` 密集填充（视觉顺序与 DOM 可能不一致）。gap 同 Flex。

### 3. 对齐

**justify = 水平，align = 垂直，place 先垂直后水平**（与 Flex 的直觉相反）。整网格对齐用 justify-content / align-content（写在容器）；项目在单元格内对齐用 justify-items / align-items（容器，管全部）与 justify-self / align-self（项目，管单个）。

### 4. 应用

Grid 行轨道整行对齐，**实现不了瀑布流**；瀑布流用多列布局。

## 八、其他布局方式

### 1. 普通流与文档流

普通流 = 默认布局：块级纵向排列、行内横向排列。脱流三途径：浮动（文字环绕）、absolute/fixed、flex/grid 子项。脱流后不撑开父元素高度、不影响兄弟排布。

### 2. 浮动 float

float 让元素脱流并贴边排列，行内内容环绕它；**导致父元素高度塌陷**。清除浮动：父元素开 BFC（推荐 `display: flow-root`），或经典 clearfix：`.clearfix::after { content: ""; display: block; clear: both; }`。

### 3. 多列布局

`column-count` 列数、`column-width` 列宽（二选一或 count 为上限）、`column-gap`、`column-rule` 分隔线。按列填充、列高度独立——配合 `break-inside: avoid` 防截断即纯 CSS 瀑布流；局限是阅读顺序纵向。

## 九、响应式布局

### 1. 媒体查询 @media

`@media (max-width: 768px) { ... }`，特性：max/min-width、orientation、prefers-color-scheme、hover。**min-width 移动优先**是推荐策略。断点按内容何时挤坏来定，不追机型。

### 2. viewport 与移动端适配

前提是 `<meta name="viewport" content="width=device-width, initial-scale=1.0">`。**rem 方案**：根字号挂视口（如 `calc(100vw / 7.5)`），尺寸用 rem，配 postcss-pxtorem 自动换算。**vw 方案**：设计稿 px ÷ 7.5 直接写 vw，无 JS，当前更推荐。

### 3. 响应式单位与函数

%、em/rem、vw/vh（及 vmin/vmax）。`calc()` 混合单位运算（运算符两侧留空格）。`clamp(min, 首选, max)` 流式字号：`font-size: clamp(14px, 2vw, 20px)`。

## 十、过渡、变换与动画

### 1. transition 过渡

四属性：property（默认 all）、**duration 必写**、timing-function（ease/linear/cubic-bezier/steps）、delay。只在属性值变化时触发。**display: none ↔ block 无法过渡**（不在渲染树），淡入淡出用 opacity + visibility；height: auto 不能过渡。

### 2. transform 变换

translate / scale / rotate / skew，3D 配 perspective；transform-origin 默认中心。**不触发重排**（性能优于改 top/left），是动画首选。副产品：absolute + translate(-50%,-50%) 未知宽高居中；祖先有 transform 时 **fixed 失效**。

### 3. animation 动画

@keyframes 定义关键帧（from/to 或百分比）。animation-name/duration/timing-function/delay + iteration-count（infinite）、direction（alternate 往返）、**fill-mode: forwards（保持结束帧）**、play-state（paused）。与 transition 区别：transition 需触发、只有两状态；animation 自动播放、多关键帧、可循环可暂停、有 animationend 事件。简单状态反馈用 transition，复杂动效用 animation。

## 十一、CSS 变量与进阶特性

### 1. 自定义属性 CSS 变量

`--primary: #409eff;` 定义（:root 全局），`var(--primary, 兜底值)` 使用。**层叠继承**：任意元素重定义即影响其子树（局部换肤）。运行时生效，JS 可 `style.setProperty` 修改——与 Sass 变量（编译期替换为死值）的本质区别，主题切换靠它。

### 2. CSS3 新特性汇总

按线记：选择器增强与结构伪类；圆角/阴影/渐变/rgba；transition/transform/animation；flex/grid/多列；媒体查询与视口单位；@font-face；calc()；自定义属性。

### 3. 伪类与伪元素

单冒号伪类（状态的已有元素），双冒号伪元素（制造的虚拟元素）。高频：`:nth-child(an+b)` 斑马纹、`:not(:last-child)` 间隔、`:focus-within` 容器高亮、`:has()` 父选择器（`form:has(input:invalid)`）；`::before/::after` 必须有 content、默认行内，用于 clearfix、装饰、图标。

## 十二、CSS 常见面试问题

### 1. 水平垂直居中的实现方式

首选 flex：`justify-content: center + align-items: center`；或 grid `place-items: center`。定位法一：absolute + `top/left: 50%` + `translate(-50%, -50%)`（**不需已知宽高**）。定位法二：absolute + inset: 0 + `margin: auto`（**需已知宽高**）。传统：text-align + line-height、table-cell + vertical-align。

### 2. 重排（回流）与重绘

重排 = 几何变化重算布局（开销大），重绘 = 仅外观变化（开销小）；重排必重绘，反之不然。优化：**动画用 transform/opacity（合成层，不重排）**；批量改 DOM（文档片段、切类名）；读写分离避免强制同步布局；will-change 提升合成层。

### 3. 隐藏元素的方式与区别

| | display: none | visibility: hidden | opacity: 0 |
|---|---|---|---|
| 占空间 | 否（脱流） | 是 | 是 |
| 触发 | 重排 | 重绘 | 重绘（合成层） |
| 可交互 | 否 | 否 | **是（配 pointer-events: none）** |
| 子元素恢复 | 不能 | 可以（设 visible） | 不透明度无法超过父级 |
| 过渡 | 不行 | 可以 | 可以 |

### 4. Flex 与 Grid 的区别与选型

**Flex 一维**（一次一行或一列）、内容驱动，适合组件内部（导航、工具条、内容流）；**Grid 二维**（行列同时）、布局驱动，适合页面骨架、卡片矩阵。常态是混用：Grid 搭骨架、Flex 排内容。注意同名对齐属性含义互换（Flex 的 justify 是主轴，Grid 的是水平轴）。

### 5. CSS 选择器优先级如何计算（见 [[#一、CSS 基础]]）

---

上级：[[web前端开发/HTML+CSS/HTML+CSS|HTML+CSS]]

## 笔记

- [[web前端开发/HTML+CSS/CSS|CSS（详细版）]]
- [[web前端开发/HTML+CSS/八股文问题|八股文问题]]
- [[web前端开发/HTML+CSS/颜色|颜色]]
