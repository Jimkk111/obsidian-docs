---
title: CSS
source: https://www.yuque.com/mook-mvqcp/wc9wsu/lng0rgnbdbfm366v
created: 2026-05-25
updated: 2026-09-08
tags:
  - HTML+CSS
---

> [!info] 目录说明
> 目录结构参照 MDN Web Docs 的 [CSS 学习区](https://developer.mozilla.org/zh-CN/docs/Learn_web_development/Core/Styling_basics) 与 [CSS 参考](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference)，并参考《CSS 权威指南（第 4 版）》补齐面试高频主题。

## 一、CSS 基础

### 1. 引入方式

CSS 有三种引入方式。**行内样式**是把样式直接写在元素的 `style` 属性里，如 `<div style="color: red">`，它的优先级最高（仅次于 `!important`），但样式与结构耦合、无法复用，只在动态设置单元素样式（JS 操作 `element.style`）或极特殊的场景下使用。**内嵌样式表**是把 CSS 写在 HTML 的 `<style>` 标签内，适合单页面的小样式，但多页面无法共享。**外部样式表**是通过 `<link rel="stylesheet" href="style.css">` 引入独立的 .css 文件，这是推荐方式：浏览器可以缓存它，多个页面可以复用，结构与样式彻底分离。

与 `<link>` 容易混淆的是 `@import`。它是一条 CSS 语法（必须写在样式表的最前面，其他规则之后写的会被忽略），用于在一个样式表里导入另一个样式表，如 `@import url("other.css")`。两者的区别主要有三点：其一，`<link>` 是 HTML 标签，浏览器解析到它就会**并行**加载 CSS，而 `@import` 要等浏览器解析到包含它的那个样式表之后才发起对目标文件的请求，是**串行**加载，容易造成页面闪一下无样式的状态（FOUC）；其二，多个 `<link>` 之间的顺序由源码顺序决定，而 `@import` 的优先级低于同文件中的其他规则；其三，`<link>` 可以配合 `rel="preload"` 做预加载，媒体查询等能力也更丰富。实践中外部引入一律用 `<link>`，`@import` 主要出现在预处理器（如 Less/Sass）编译前的源码里。

### 2. 语法与规则集

CSS 的基本组成单元是**规则集**（rule set），由选择器和声明块构成：选择器定位元素，声明块写在一对花括号里，每条声明是 `属性: 值` 的形式，以分号结尾。例如 `h1 { color: red; font-size: 24px; }`。注释只有一种写法 `/* ... */`，没有 `//` 单行注释。

除普通规则外，CSS 中还有以 `@` 开头的**@规则**（at-rules），用来表达"顶层指令"：`@media` 媒体查询、`@import` 导入样式、`@font-face` 定义自定义字体、`@keyframes` 定义动画关键帧、`@supports` 特性检测、`@page` 打印分页等。理解规则集和 @规则这两类结构，就理解了 CSS 文件的组织方式。

### 3. 选择器与优先级

**选择器决定"样式写给谁"**，是 CSS 的核心，这里尽量列全。

基础选择器有五种：通配选择器 `*` 匹配所有元素；类型选择器（标签选择器）`p` 按标签名匹配；类选择器 `.box` 按 class 匹配，一个元素可以有多个类、一个类可以作用于多个元素，是开发中最主要的选择器；ID 选择器 `#nav` 按 id 匹配，页面中应当唯一；属性选择器按属性匹配，写法从简单到复杂有七种——`[disabled]`（存在该属性即匹配）、`[type="text"]`（完全等于）、`[class~="btn"]`（多词空格分隔的值中包含该词，等价于 `.btn`）、`[lang|="zh"]`（值等于 zh 或以 zh- 开头）、`[href^="https"]`（以某字符串开头）、`[href$=".pdf"]`（以某字符串结尾）、`[title*="警告"]`（包含某字符串）。

组合器用来表达元素之间的关系：**后代组合器**用空格表示，`div p` 匹配 div 内任意层级的 p；**子组合器** `>` 只匹配直接子元素，`ul > li` 不会命中嵌套更深层的 li；**相邻兄弟组合器** `+` 匹配紧随其后的同级元素，`h2 + p` 选中紧跟在 h2 后的第一个 p；**通用兄弟组合器** `~` 匹配之后所有满足条件的同级元素，`h2 ~ p` 选中 h2 后面全部的 p。选择器还可以串联叠加提高精度，如 `div.box#main:hover`。

伪类以单冒号开头，表示元素的**状态或结构位置**，可细分为几族。动态/交互伪类：`:link`（未访问链接）、`:visited`（已访问）、`:hover`（悬停）、`:active`（按下）、`:focus`（聚焦）与 `:focus-within`（自身或后代聚焦）、`:focus-visible`（由键盘聚焦时才命中）。UI 状态伪类：`:checked`、`:enabled`、`:disabled`、`:required`、`:optional`、`:valid`、`:invalid`、`:placeholder-shown`。目标与语言伪类：`:target`（URL 片段指向的锚点元素）、`:lang(zh)`。结构伪类：`:root`（文档根元素）、`:empty`（无子元素）、`:first-child` / `:last-child` / `:only-child`、`:first-of-type` / `:last-of-type` / `:only-of-type`、`:nth-child(an+b)`（可用 `odd` / `even` 或公式如 `2n+1`）、`:nth-last-child()`（从后往前数）、`:nth-of-type()` / `:nth-last-of-type()`。这里要分清 `:nth-child()` 与 `:nth-of-type()`：前者在**所有兄弟**中数位置（不看标签类型，类型不符则不匹配），后者只在**同类型标签**的兄弟中数位置。逻辑伪类是较新的成员：`:not(选择器)` 取反、`:is()` 匹配列表中任意一个（可简化长选择器）、`:where()` 与 `:is()` 相同但**优先级为零**、`:has(相对选择器)` 根据"是否包含某元素"来匹配（父选择器），如 `div:has(img)`。

伪元素以双冒号开头，表示**创建一个 DOM 中不存在的虚拟元素**或选中元素的某个部分：`::before` / `::after` 在元素内容前后插入虚拟盒子（必须有 `content` 属性，默认行内，是 clearfix、装饰图形、图标的最常用手段）、`::first-line`（首行）、`::first-letter`（首字母）、`::selection`（被用户选中的文本）、`::placeholder`（输入框占位文字）、`::marker`（列表项的项目符号）、`::backdrop`（全屏或弹层的背景遮罩）。伪类与伪元素的根本区别：**伪类选中的是"处于某状态的已有元素"，伪元素"制造"了一个本来不存在的元素**。

**优先级（specificity，也译特异性）**决定多条规则命中同一元素时谁生效，它按"四位数字"比较：行内样式为 1-0-0-0；ID 选择器贡献第二位的 1；类选择器、属性选择器、伪类各贡献第三位的 1；类型选择器和伪元素各贡献第四位的 1。通配符 `*`、组合器（`>`、`+`、`~`、空格）以及 `:where()` 不贡献任何位数。比较时从左到右逐位比大小，例如 `#nav .item`（0-1-1-0）永远大于 `div ul li a.active`（0-0-2-3），无论后者写多长。`!important` 的优先级凌驾于一切普通规则之上，完整的高低顺序是：`!important` 的行内样式 > `!important` 的作者样式 > 普通行内样式 > 普通作者样式；`!important` 滥用会破坏层叠的可预测性，只建议在覆盖第三方库样式等无奈场景使用。

当优先级完全相同时，按**源码顺序**后写的生效——这就是为什么把状态伪类写成 `:link :visited :hover :active`（爱恨法则 LVHA）的固定顺序：`:hover` 必须写在 `:visited` 之后才能在已访问链接上生效。此外要注意**继承的属性没有优先级**：继承来的值会被任何直接命中的规则覆盖，即使那条规则只有一个通配符。

与层叠并列的还有**继承**：默认继承的多是与"文字外观"相关的属性，如 color、font 系列、line-height、text-align、visibility；盒模型属性（width、margin、border 等）默认不继承。四个通用属性值可以显式控制继承行为：`inherit`（强制继承父值）、`initial`（重置为规范初始值）、`unset`（可继承属性等同 inherit，不可继承属性等同 initial）、`revert`（回退到浏览器默认样式表）。

### 4. 值与单位

长度单位中，`px` 是 CSS 像素（逻辑像素），它不是物理像素，在高分屏上一个 CSS 像素对应多个物理像素。`em` 是相对单位，用在 `font-size` 上表示**父元素**的字体大小，用在其他属性上表示**自身**字体大小的倍数，这个双重身份是 em 的经典考点。`rem` 统一相对根元素（html）的字体大小，规避了 em 的级联累积问题，是移动端 rem 适配方案的基石。`vw` / `vh` 分别是视口宽度和高度的百分之一，另有 `vmin` / `vmax` 取视口短边/长边。百分比 `%` 的参照对象因属性而异：width 相对父元素宽度、line-height 相对自身字体大小、margin/padding 相对**包含块的宽度**（垂直方向也是宽度）。颜色值详见 [[web前端开发/HTML+CSS/颜色|颜色]]。其余常见值类型还有角度 `deg`、时间 `s` / `ms`、无单位数字（line-height 推荐、flex-grow、z-index）以及关键字。

## 二、盒模型

### 1. content、padding、border、margin

盒模型是对页面元素结构的抽象：每个元素被表示为一个盒子，从内到外依次是**内容区 content**（元素的实际内容，width/height 默认描述的就是它）、**内边距 padding**（内容与边框之间的空隙，背景色会延伸到 padding）、**边框 border**（围绕内容和 padding 的线条）、**外边距 margin**（盒子与相邻元素之间的透明间隔，永远不显示背景）。

使用上的几个要点：padding 不能为负值，margin 可以是负值（负 margin 常用来微调重叠和布局）；`margin: 0 auto` 可以让块级元素在父元素内水平居中，但垂直方向的 auto 不会居中（这是面试常客，垂直居中要靠 flex、定位或 transform）；四个值的简写顺序是上、右、下、左（顺时针），三个值是上、左右、下，两个值是上下、左右。

### 2. box-sizing 与元素宽高

`box-sizing` 决定 width 和 height 这两个属性到底"描述盒子的哪一部分"。**content-box**（默认值，标准盒模型）下，`width` 只表示内容区的宽度，元素实际占据的宽度 = width + 左右 padding + 左右 border，再加上外部的 margin。**border-box**（怪异盒模型）下，`width` 表示内容 + padding + border 的总宽度，改 padding 不会把盒子撑大，内容区自动被压缩。

两种模式各有所长，但 border-box 更符合直觉——"我设的宽就是盒子总宽"，所以现代项目普遍全局设置 `*, *::before, *::after { box-sizing: border-box; }`。要澄清一个误区：这里说的"怪异盒模型"与浏览器"怪异模式"是两回事，标准模式渲染的页面同样可以给元素设置 border-box。另外，更换 box-sizing 会改变元素尺寸的计算方式，但 margin 永远在盒子尺寸之外，不受它影响。

### 3. margin 合并（外边距塌陷）

**外边距合并**指块级元素在垂直方向上的 margin 会"叠在一起"而不是相加，只发生在普通文档流中相邻的块级盒子之间，水平方向永远不会合并。它有三种发生场景：**相邻兄弟**之间，上元素的下 margin 和下元素的上 margin 取两者中的较大值（正 margin 取大，负 margin 取小，正负混合则相加）；**父元素与第一个/最后一个子元素**之间，子元素的 margin-top 会"穿透"父元素，表现为父元素被顶下去——这是最反直觉的一种，给父元素设 margin-top 想拉开距离往往无效；**空的块级元素**自身的上下 margin 也会合并成一个。

理解了成因，解决办法自然围绕"隔断合并条件"展开。最常用的是给父元素**开启 BFC**（如 `overflow: hidden` 或语义化更好的 `display: flow-root`），BFC 容器与内部子元素的 margin 不会合并，同时也能包住子元素的高度；或者在父子之间插入分隔——给父元素加 border 或 padding-top，使子元素的 margin 无法再与外界接触；改用 **flex 或 grid 布局容器**是现代做法，它们的子项之间 margin 不会合并。兄弟元素之间的合并如果确实需要避免（比如想要 margin 相加），也可以给其中一个元素开启 BFC 或改用行内块/flex 排布，但多数情况下"取较大值"本来就是期望行为。

### 4. BFC 块级格式化上下文

**BFC（Block Formatting Context，块级格式化上下文）**是一块独立的渲染区域，可以理解为"内部布局自成一体、与外界互不干扰的隔离舱"：内部元素的布局不影响外部，外部也影响不到它。

创建 BFC 的方式包括：根元素 `html`；浮动元素（float 非 none）；绝对定位元素（position 为 absolute 或 fixed）；`display: flow-root`（专门为此设计的属性，无副作用，推荐）；`display: inline-block`、`table-cell`、`table-caption`、`flex`、`grid`、`inline-flex`、`inline-grid`；`overflow` 非 visible（hidden、auto、scroll）。

BFC 的价值在于它的几条布局规则，对应四个经典应用场景。第一，**BFC 容器会包含内部的浮动元素**——浮动元素脱离文档流后父元素高度塌陷，把父元素变成 BFC 就能重新包住浮动的子元素，这是清除浮动的原理。第二，**BFC 区域不会与浮动元素重叠**——利用这一点可以做出"旁边有浮动图片、文字不环绕"的自适应两栏布局：给右侧文字块开启 BFC，它就会避开左侧的浮动列。第三，**BFC 容器与外部元素的 margin 不合并**，可以隔离父子、空元素的 margin 塌陷。第四，BFC 内部的盒子按正常规则排列，内部塌陷问题不会"泄漏"到外部。面试时按"是什么（独立渲染区域）→ 怎么触发 → 有什么特性/能解决什么问题"的顺序回答即可。

## 三、文本与字体

### 1. 字体属性 font

`font-family` 设置字体栈，浏览器从左到右依次回退：`font-family: "Helvetica Neue", Arial, "PingFang SC", "Microsoft YaHei", sans-serif;`。字体名有空格要加引号，中文字体建议同时写英文名；栈的末尾放一个通用字体族（serif 衬线、sans-serif 无衬线、monospace 等宽）作为最后防线。中英文字体混排时把英文字体写在前面，因为浏览器按顺序匹配，英文字体通常不含中文字形，会自动回退到后面的中文字体。

`font-size` 设置字号，常用 rem / px；`font-weight` 设置字重，取 100~900 的数字或 normal（400）、bold（700），实际显示效果取决于字体是否提供了对应字重，否则浏览器近似合成；`font-style` 设置斜体（italic / oblique / normal）。简写 `font` 的格式是 `font: style weight size/line-height family`，其中 **size 和 family 必须写**，其余可省略，且 style 和 weight 必须写在 size 之前。

`@font-face` 允许网站加载自定义字体文件：`@font-face { font-family: "MyFont"; src: url("myfont.woff2") format("woff2"); font-display: swap; }`，之后就能在 font-family 中使用。格式上 woff2 体积最小应优先提供；`font-display` 控制字体加载期间的表现，`swap` 表示先显示回退字体、字体就绪后替换（避免文字长时间不可见的 FOIT 问题）。图标字体（iconfont）也是基于 @font-face 实现的。

### 2. 文本属性 text

`color` 设置文字颜色。`text-align` 控制块级容器内部行内内容的水平对齐，取值 left / center / right / justify（两端对齐）——注意它作用于**块级元素**、影响的是其内部的行内内容，而不是元素自身在父元素中的位置。`line-height` 设置行高，推荐使用**无单位数字**（如 1.5，表示自身 font-size 的 1.5 倍），因为它会被后代按各自的字号重新计算继承，而带单位 px 的行高会原样继承导致换行错乱；单行文字在容器内垂直居中的经典技巧就是令容器的 height 等于 line-height。

`text-decoration` 设置装饰线，取值 underline（下划线）、line-through（删除线）、overline、none（去掉 a 的默认下划线就靠它）。`text-indent` 设置首行缩进，`letter-spacing` / `word-spacing` 控制字/词间距，`white-space` 控制空白与换行处理（nowrap 强制不换行、pre 保留空白格式），`word-break` / `overflow-wrap` 处理长单词的换行。

文本溢出省略是最常考的组合技。**单行省略**三件套：`overflow: hidden; white-space: nowrap; text-overflow: ellipsis;`——先不换行、再裁掉溢出、最后用省略号标记。**多行省略**用 WebKit 私有属性：`display: -webkit-box; -webkit-box-orient: vertical; -webkit-line-clamp: 3; overflow: hidden;`，限制为 3 行后超出部分显示省略号，兼容性目前已经很好。

## 四、背景、边框与视觉效果

### 1. background 系列

`background-color` 设置背景色，`background-image` 设置背景图，二者可叠加，图片未铺满或未加载时露出颜色。CSS 允许**多层背景**：`background-image: url(a.png), url(b.png)` 写多个值，逗号分隔，**先写的图层在上层**。渐变本质上也是 image 类型：`linear-gradient(to right, red, blue)`、`radial-gradient(circle, ...)`、`conic-gradient(...)`，可直接用于 background-image，能替代很多切图。

`background-repeat` 控制平铺（repeat / no-repeat / repeat-x / repeat-y，另有 space 拉开不裁剪、round 缩放铺满）。`background-position` 控制位置，可用关键字、百分比或长度；用百分比时有个反直觉的规则：`50% 50%` 是图片中心对准容器中心，因为百分比的含义是"图片的 x% 点对准容器的 x% 点"。`background-size` 控制尺寸，两个关键字最常用：`cover` 等比放大到**完全覆盖**容器（可能裁剪图片），`contain` 等比缩放到**完整显示**（可能留白）。

`background-attachment` 控制背景随什么滚动：scroll 随内容滚动、fixed 固定于视口（可做视差效果）、local 随元素内容滚动。`background-origin` 决定背景图定位的基准盒（border-box / padding-box / content-box），`background-clip` 决定背景绘制到哪一层为止，其中 `background-clip: text` 配合透明文字可以做出"文字镂空显示背景图"的效果。简写属性 `background` 可以一次写完，注意 position 和 size 同用时中间要用斜杠分隔：`background: url(bg.jpg) center/cover no-repeat;`。

### 2. 边框与圆角

`border` 的三要素是宽度、样式、颜色，**样式必填**（不写 style 边框不存在，因为默认值 none）：`border: 1px solid #ccc;`。可以按边单独设置 border-top / border-right 等，三角形、梯形等图形就是利用"边框交界处是斜切"的原理，把元素宽高设为 0、只留三边 border 画出来的。

`border-radius` 设置圆角，四个值的顺序是左上、右上、右下、左下（顺时针）。设为 50% 可以把正方形变圆、矩形变椭圆；它还支持斜杠语法 `border-radius: 50% / 30%`，分别指定每个角的水平半径和垂直半径，用于做叶形、蛋形等不规则圆角。

`border-image` 用图片作为边框，配合 `border-image-slice` 切分九宫格、`border-image-repeat` 控制填充方式，使用频率不高。**outline 与 border 的区别**是常考对比：outline 绘制在边框外缘，**不占据布局空间**（可能覆盖周围内容），不能单独设置某一边，通常用在 `:focus` 状态做键盘导航的焦点轮廓；border 是盒模型的一部分，占据空间、可分边设置。

### 3. 阴影

`box-shadow: 水平偏移 垂直偏移 模糊半径 扩散半径 颜色 inset;`。前两个偏移可以是负值；模糊半径越大越柔和；扩散半径把阴影整体放大或缩小（负值收缩），配合 0 偏移可以画描边或光晕；`inset` 把阴影翻转到盒子内部。阴影不占布局空间，多层阴影用逗号分隔叠出更立体的效果。`text-shadow: 水平 垂直 模糊 颜色;` 给文字加阴影，比 box-shadow 少了扩散半径和 inset。

### 4. overflow 与裁剪

`overflow` 决定内容溢出盒子时怎么处理：`visible`（默认，溢出可见）、`hidden`（裁掉，滚动不可见）、`scroll`（始终显示滚动条，哪怕内容不溢出）、`auto`（溢出才出现滚动条）、`clip`（类似 hidden 但连编程滚动都不允许）。可以分轴设置 overflow-x / overflow-y；注意当一轴设置 hidden 而另一轴是 visible 时，visible 会被浏览器改算为 auto。

两个连带的知识点：`overflow` 非 visible 会使元素**创建 BFC**，这是 overflow: hidden 能清除浮动、隔离 margin 塌陷的原理；`overflow: hidden` 还常与 `text-overflow: ellipsis`、`-webkit-line-clamp` 搭配实现文本省略（见 [[#三、文本与字体]]）。但要注意 hidden 会连带裁掉定位溢出的子元素和阴影，sticky 定位的父级若 overflow 非 visible/clip 会导致 sticky 失效。

### 5. filter 滤镜

filter 属性是滤镜属性，其取值 `blur()` 表示模糊半径，可以搭配 inset 使用（作用是把元素拉大到大于背景，因为 blur() 需要参考附近像素计算模糊，如果大小和背景一样大，靠边界的区域就没有模糊效果），打造盖在背景上的模糊层。

## 五、定位与层叠

### 1. position 定位

position 决定元素如何定位，配合 top / right / bottom / left（统称 inset 属性）使用，五个取值的行为差异是面试重点。

`static` 是默认值，元素处于正常文档流，offset 属性无效。`relative` 相对定位：元素**相对自身原本的位置**偏移，占据的空间保留（不脱离文档流），原位置的"影子"还在。它最常见的用途不是偏移自己，而是给 absolute 子元素充当**定位基准**。

`absolute` 绝对定位：元素**脱离文档流**，相对**最近的非 static 祖先**定位，找不到这样的祖先就相对初始包含块（通常是视口大小的 html 区域）。因为脱流，它不再占据空间、不影响兄弟元素，宽度默认收缩为内容宽度——这也意味着绝对定位实现了"自适应内容宽"的居中难点（见 [[#十二、CSS 常见面试问题]]）。

`fixed` 固定定位：脱离文档流，相对**视口**定位，滚动时纹丝不动，用于吸顶按钮、悬浮客服等。它有一个重要失效场景：**祖先元素设置了 transform、filter、perspective 或 will-change 时，fixed 会改为相对该祖先定位**（因为它们创建了包含块），滚动吸附随之失效，这是实际开发中反复出现的坑。

`sticky` 粘性定位：介于 relative 和 fixed 之间，元素在正常流中，滚动未到达阈值时表现如 relative，到达指定阈值（如 `top: 0`）后"吸附"在原地如 fixed。它不脱离文档流，必须至少指定一个方向的 offset 才会生效；父容器若设置 overflow: hidden / auto / scroll，sticky 会失效，且吸附范围不会超出父元素。

### 2. inset 属性

inset 属性很实用，是 top、bottom、right 和 left 的简写，可以控制定位元素的大小。

### 3. z-index 与层叠上下文

`z-index` 控制同一层叠平面内元素的绘制顺序，数值大的在上层。它的生效条件常被忽略：**只对 position 非 static 的元素以及 flex/grid 项**有效，对普通流元素无效。

**层叠上下文**（stacking context）是理解层叠的关键。它是元素在三维上的"图层"：每个层叠上下文是一个独立单元，**内部子元素的 z-index 只能在本上下文内比较，整个上下文作为整体参与外部的层叠**。这带来一个经典结论：父级 A（z-index: 1）和父级 B（z-index: 2）是兄弟，A 内部的子元素无论 z-index 多大，都盖不过 B 及其子元素，因为"图层 A 整体"被压在"图层 B"之下。

创建层叠上下文的条件很多：根元素；position 为 absolute/fixed 且 z-index 非 auto 的元素；`opacity` 小于 1 的元素；设置了 transform、filter、perspective、clip-path、will-change、isolation: isolate 的元素；flex/grid 容器中 z-index 非 auto 的子项等。这意味着 z-index 失效的排查思路通常是：看目标元素是否 position 非 static（或 flex/grid 项），再沿着祖先链找是否有某个祖先意外创建了层叠上下文，把它整个"封"在了低层级。

在不指定 z-index 时的默认绘制顺序也值得记：背景和边框最先绘制，然后是负 z-index 的子上下文、普通流块级盒、浮动盒、行内盒，最后是 z-index 为 0/auto 的定位元素和正 z-index 元素——这解释了"为什么负 z-index 的元素会藏在父元素背景之后"。

## 六、Flex 布局

### 1. 容器属性

给容器设置 `display: flex`（块级 flex 容器）或 `display: inline-flex`（行内块 flex 容器）即开启 Flex 布局。开启后容器内的布局逻辑整体改变：子元素（称为**项目**，flex item）的 float、clear、vertical-align 失效，子元素之间的 margin 不再合并。

容器上最重要的概念是**主轴与交叉轴**：主轴由 `flex-direction` 决定（`row` 默认水平向右、`row-reverse` 水平向左、`column` 垂直向下、`column-reverse` 垂直向上），与主轴垂直的就是交叉轴。后续所有对齐属性都是围绕这两根轴定义的，轴一变，属性含义跟着变——这是 flex 难记的根源。

容器属性共六个加 gap：`flex-wrap` 控制换行（`nowrap` 默认不换行、项目会被压缩；`wrap` 换行，第一行在上；`wrap-reverse` 换行且第一行在下），`flex-flow` 是 direction 和 wrap 的简写。`justify-content` 控制**项目在主轴上的对齐**：flex-start（默认，起点对齐）、flex-end、center、`space-between`（两端对齐，中间等分）、`space-around`（每个项目两侧间距相等，所以中间的间距是两端的两倍）、`space-evenly`（所有间距完全相等）。`align-items` 控制**单行内项目在交叉轴上的对齐**：`stretch` 默认，项目未设高度时拉伸占满行高——这就是 flex 子项默认等高的原因；还有 flex-start、flex-end、center、baseline（按文字基线对齐）。`align-content` 控制**多行作为整体**在交叉轴上的对齐，只有换行后才有意义，单行时无效。`gap` / `row-gap` / `column-gap` 设置项目之间的固定间距，比用 margin 更干净。

### 2. 项目属性

项目上的属性有五个，核心是 flex 三件套。`flex-grow` 定义**剩余空间**的放大比例，默认 0（不放大）；设为 1 则所有此类项目均分剩余空间，按比例设置数值即按比例分配。`flex-shrink` 定义**空间不足**时的缩小比例，默认 1（等比压缩，这也是内容被压扁的默认原因），设为 0 则禁止缩小。`flex-basis` 定义项目在主轴上的基准尺寸，默认 `auto`（参照自身的 width/height 或内容），设置后优先级高于 width。

`flex` 简写按 `grow shrink basis` 顺序记忆，几个常用值必须烂熟：`flex: 1` 等于 `1 1 0%`，即忽略自身宽度、完全均分剩余空间，是"平分一栏"和"右侧自适应"的标准写法；`flex: auto` 等于 `1 1 auto`，在自身内容宽度基础上分配剩余空间；`flex: none` 等于 `0 0 auto`，完全不伸缩、保持内容原始大小，用于固定不缩放的元素（如固定宽度的侧栏）。

`align-self` 让单个项目覆盖容器的 align-items 设定自己的交叉轴对齐；`order` 改变项目的排列顺序（默认 0，数值小的在前，可为负）——它只改视觉顺序不改 DOM 顺序，配合媒体查询可以低成本实现"移动端把主内容排到前面"。

### 3. 常见布局场景

**水平垂直居中**的 flex 方案只有两行：容器 `display: flex; justify-content: center; align-items: center;`，无惧子元素未知宽高，是现代首选。

**左侧固定宽度、右侧自适应**：左栏固定 width（配 `flex: none` 防缩放），右栏 `flex: 1`。**等分布局**（如九宫格、均分按钮组）：每个子项 `flex: 1`，默认 stretch 顺带实现等高。**圣杯布局/双飞翼布局**：float 年代需要负 margin 和 padding 补丁的经典三栏布局，用 flex 只需中间栏 `flex: 1`、两侧栏固定宽度，配合 `order` 还能自由调整 DOM 与视觉的顺序，代码量骤减。

**粘性页脚**（内容不足一屏时页脚仍贴底）：外层容器 `min-height: 100vh; display: flex; flex-direction: column;`，内容区 `flex: 1`，页脚自然被推到底部。此外，让某个元素"吸底/靠右推走"的技巧是给它加 `margin-left: auto`（主轴起点在左时），利用 auto margin 吞掉全部剩余空间的特性，比嵌套空容器优雅得多。

## 七、Grid 布局

### 1. 基础概念

理解 Grid 先要建立一套空间词汇。**网格线**（grid line）是划分行列的分界线，一个有 n 列的网格有 n+1 条列网格线，网格线按顺序编号，也是项目定位的坐标。两条相邻的平行网格线围成一个**网格轨道**（grid track），也就是一行或一列。一条列轨道和一条行轨道交叉形成**网格单元**（grid cell），它是 Grid 里最小的排布单位，相当于表格中的一个格子。任意四条网格线围成的封闭区域是**网格区域**（grid area），它可能由多个单元拼成，是项目跨行跨列占据的空间。

视角转到元素上：开启 `display: grid` 的元素是**网格容器**（grid container），容器的直接子元素是**网格项**（grid item）——注意只有直接子元素才算网格项，更深层的后代不受网格管理。

### 2. 样式和函数、关键字

容器上用 `display: grid` 开启网格布局，然后用 `grid-template-rows` 和 `grid-template-columns` 定义显式的行与列，值之间空格分隔即声明了轨道数量，如 `grid-template-columns: 100px 1fr 100px` 定义了三列。对应地，`grid-auto-rows` 和 `grid-auto-columns` 定义**隐式轨道**：当项目数量超出显式定义的行列、或项目被定位到显式网格之外时，网格会自动生成额外的行列，这些"计划外"的行列尺寸就由这两个属性控制。

轨道尺寸有三件套语法值得专门记。`fr` 是 Grid 特有的**剩余空间比例单位**，`1fr 2fr` 表示把剩余空间按 1:2 分配。`repeat()` 用于声明多个尺寸相同的轨道，如 `repeat(3, 1fr)` 等价于 `1fr 1fr 1fr`，第一个参数还可以换成 `auto-fill` 或 `auto-fit`——当**不确定一行能放几个**时，浏览器按指定尺寸自动塞满一行，这是响应式卡片网格的黄金搭配：`repeat(auto-fill, minmax(200px, 1fr))` 一行代码就能实现"宽度够就多放一列、每列最少 200px"。`minmax(min, max)` 声明一个尺寸范围，轨道会在两者之间伸缩，常与 auto-fill/fr 组合使用。

项目上用 `grid-row` 和 `grid-column` 声明跨行、跨列，值可以写"起始线 / 结束线"（如 `grid-column: 1 / 3` 表示从 1 号线到 3 号线、跨两列），也可用 `span n` 表示"跨 n 个轨道"（如 `grid-column: span 2`）。容器上 `grid-auto-flow` 决定自动放置算法的排列方向（row 默认按行、column 按列），加上 `dense` 关键字会启用**密集填充**，让后面的项目回填前面留下的空位——代价是视觉顺序与 DOM 顺序可能不一致。最后 `gap` / `row-gap` / `column-gap` 设置轨道间距，与 Flex 中用法相同。

### 3. 对齐

Grid 的对齐体系与 Flex 同名但含义不同，先记总原则：**justify 表示水平方向，align 表示垂直方向**（与 Flex 中 justify 是主轴的直觉相反），`place` 是两者的简写，**先垂直后水平**（如 place-items: center start）。

按作用范围分两层。第一层是**整个网格在容器内的对齐**：justify-content 和 align-content 写在容器上，控制整体网格（当网格总尺寸小于容器时才有意义）在容器里的位置。第二层是**项目在单元格内的对齐**：justify-items 和 align-items 写在容器的选择器上，统一控制所有网格项在自己单元格内的对齐方式；如果只想调整某一个项目，就用写在项目选择器上的 justify-self 和 align-self，只作用这一个单元格。

### 4. 应用

Grid 与 Flex 的选型见 [[#十二、CSS 常见面试问题]]。一个常被误解的点：Grid 布局实现不了瀑布流——瀑布流的特征是各列高度独立增长、项目按列填充，而 Grid 的行轨道是整行对齐的，无法让同一行的项目高度参差。瀑布流应该用**多列布局**实现，或者用 JS 计算（如 Masonry 方案）。

## 八、其他布局方式

### 1. 普通流与文档流

**普通流**（normal flow，也常被叫做文档流）是浏览器默认的布局方式：块级元素从上到下逐个排列、独占一行，行内元素和行内块元素从左到右排列、放不下就换行，行盒的高度由 line-height 决定。所谓"**脱离文档流**"，就是元素不再按这套规则占位，主要有三种途径：**浮动**（float，脱流但文字会环绕它）、**绝对定位**（absolute / fixed，完全脱流）以及脱离常规的布局上下文（flex/grid 的子项不再参与普通流排布，但仍受容器管理）。

理解脱流对调试的意义在于：脱流元素不再撑开父元素高度（导致高度塌陷）、不再影响兄弟元素排布、margin 不再与普通流元素合并。日常排布中的"莫名其妙换行""父元素高度为 0"，多数都能追溯到某个元素意外脱流。

### 2. 浮动 float

浮动的历史使命是**文字环绕图片**，后来在 flex 普及之前被广泛用于多栏布局，如今主要用于环绕效果和少量特殊排布，页面骨架应交给 flex/grid。

`float: left / right` 让元素脱离文档流，向左或向右移动，直到碰到容器边缘或**另一个浮动元素**的边缘停下；多个浮动元素会像"贴"一样依次排开，一行放不下则折行。浮动元素的两个核心特性：其一，它脱离了文档流但**行内内容会环绕它**（这正是文字环绕的来源）；其二，浮动会**导致父元素高度塌陷**——子元素全部浮动时父元素高度变为 0，背景和边框"消失"。

**清除浮动**（准确说是清除浮动带来的影响，解决高度塌陷）有两类主流做法。第一类是**开启父元素 BFC**：BFC 会包含内部浮动，可用 `overflow: hidden`，但最推荐语义化、无副作用的 `display: flow-root`——它是专门为"创建无副作用的 BFC"设计的属性。第二类是**清除法**：在浮动元素之后用 `clear: both` 阻止后续元素受浮动影响，实际开发中不添加多余标签，而是给父元素加伪元素（经典 clearfix）：`.clearfix::after { content: ""; display: block; clear: both; }`。单独加一个空标签写 clear 的方式虽然能解决，但污染了 HTML 结构，不推荐。

### 3. 多列布局

多列布局（multi-column）把一段内容自动分成多栏，是文字排版意义上的"报纸分栏"，也是实现瀑布流的标准方案。容器上用 `column-count` 声明列数，或用 `column-width` 声明理想列宽（浏览器根据可用宽度自动决定实际列数，两个属性同时写时 column-count 是上限），`column-gap` 设置列间隙，`column-rule` 在列之间画分隔线，用法类似 border（如 `column-rule: 1px solid #ccc`）。

它的分栏逻辑是**按列填充**：内容先填满第一列再流向第二列，列与列之间高度独立增长——这正是瀑布流的特征。给每个卡片项加 `break-inside: avoid` 防止一张卡片被拦腰截断到两列，配合图片不定高，就能得到纯 CSS 的瀑布流。局限也很明显：多列是按"列"从上往下读的，条目顺序是纵向的而非横向的，对新闻排版无妨，但对要求阅读顺序的列表并不合适。

## 九、响应式布局

### 1. 媒体查询 @media

媒体查询是响应式布局的开关，语法为 `@media 媒体类型 and (媒体特性) { 规则 }`。媒体类型有 `all`（默认，所有设备）、`screen`（屏幕）、`print`（打印）、`speech`（读屏器）；实际开发中主要用媒体特性：`max-width` / `min-width` 限定视口宽度、`orientation: portrait / landscape` 区分横竖屏、`prefers-color-scheme` 适配深色模式、`hover: none` 识别触屏设备。

写法上存在两种断点策略：**min-width（移动优先）**从小屏写起，逐级向上加样式，是业界推荐的默认策略，因为移动端样式往往更简单；**max-width（桌面优先）**反之。除了写在 CSS 内部，媒体查询也能用在 `<link media="...">` 上按条件加载样式表，以及 `<source media="...">` 上做响应式图片的艺术指导（见 [[web前端开发/HTML+CSS/HTML|HTML]] 的响应式图片一节）。选择断点时不要"追着具体机型写"，而是按内容在什么宽度开始"挤坏"来定断点，常见的 640 / 768 / 1024 / 1280 只是参考习惯。

### 2. viewport 与移动端适配

移动端适配的第一步永远是 `<meta name="viewport" content="width=device-width, initial-scale=1.0">`，让布局视口等于设备理想宽度（原理见 [[web前端开发/HTML+CSS/HTML|HTML]] 面试问题一节），否则后面所有方案都建立在错误的视口宽度上。

主流适配方案有两条路线。**rem 方案**：把根元素字号与视口宽度挂钩（如设计稿宽 750px 对应 `html { font-size: calc(100vw / 7.5) }`，即 1rem = 100px），页面中所有尺寸用 rem 书写，视口变化时等比缩放；手工换算繁琐，工程上用 **postcss-pxtorem** 在构建时把 px 自动转成 rem，旧时代的 lib-flexible 则是在运行时用 JS 动态设置根字号。**vw 方案**：设计稿上 750px 宽即 100vw，元素直接用 vw 书写（px 值 ÷ 7.5 得 vw），无 JS 依赖、无层级 rem 换算，是当前更受推荐的方案。两方案可结合：字体等需要"小屏别缩太小"的属性仍用 rem + 媒体查询兜底。

### 3. 响应式单位与函数

各响应式单位的分工：`%` 依赖父元素；em 级联累积、适合组件内部随字号联动的间距；rem 全局统一、适合整体缩放；vw / vh 直接绑定视口，适合全屏横幅和流式排版；另有 vmin / vmax 处理横竖屏切换。三个数学函数是响应式的利器：`calc()` 支持混合单位四则运算（运算符两侧必须有空格），如 `width: calc(100% - 200px)` 实现"侧栏固定、主区自适应"；`min()` 取小值、`max()` 取大值；`clamp(最小值, 首选值, 最大值)` 相当于三者的合体，用它做**流式字号**一行顶过去几行媒体查询：`font-size: clamp(14px, 2vw, 20px)`——视口在合理范围内时字号随 2vw 平滑变化，两端封顶。

## 十、过渡、变换与动画

### 1. transition 过渡

transition 让属性值在两个状态之间平滑变化而不是瞬间跳变，四个子属性为：`transition-property`（要过渡的属性，默认 all）、`transition-duration`（时长，**不写就没有过渡效果**）、`transition-timing-function`（缓动函数：ease 默认先快后慢、linear 匀速、ease-in / ease-out / ease-in-out，以及 steps(n) 分步和 cubic-bezier() 自定义贝塞尔曲线）、`transition-delay`（延迟）。简写时按 property duration timing-function delay 的顺序，前两个时间值中第一个总是 duration。

理解过渡的**触发条件**比记属性更重要：transition 只在属性值发生变化时生效，变化来源可以是 :hover、JS 切换类名或直接改样式。两个经典失效场景要记：一是 `display: none` 与 `block` 之间没有过渡——display 变化时元素根本不在渲染树里，没有"前后两个可插值的状态"；想淡入淡出应过渡 opacity 和 visibility（visibility 是离散属性，配合过渡时间可延迟隐藏）。二是并非所有属性都能过渡，只有存在中间值（可插值）的属性才行：颜色、长度、transform 可以，height 的 `auto` 值不行（要用 max-height 或 grid-template-rows 等技巧模拟）。

### 2. transform 变换

transform 对元素做几何变换，2D 函数包括 `translate(x, y)` 平移、`scale(x, y)` 缩放、`rotate(deg)` 旋转、`skew(x, y)` 倾斜；3D 函数如 translateZ、rotateX / rotateY 配合 perspective 产生透视立体效果。`transform-origin` 指定变换基准点，默认是元素中心，rotate 的效果常因基准点不符合预期而需要调整。

transform 最重要的特性是**不触发重排**：变换不改变元素在文档流中占据的位置，只影响最终绘制的样子，浏览器可以直接在合成层完成，性能远好于动画 top/left/width 这类布局属性——这就是"动画优先用 transform + opacity"的根本原因。它还有两个实用副产品：`translate(-50%, -50%)` 配合 absolute 定位实现未知宽高的水平垂直居中；以及祖先一旦设置 transform，fixed 定位会改为相对该祖先，常造成"fixed 失效"的 bug。

### 3. animation 动画

动画通过 `@keyframes` 定义关键帧序列，再用 animation 属性应用：`@keyframes slide { from { transform: translateX(0); } to { transform: translateX(100px); } }`，中间状态也可用百分比（`0%`、`50%`、`100%`）精确控制。animation 的子属性包括：animation-name、duration、timing-function、delay、`iteration-count`（次数或 infinite 无限循环）、`direction`（alternate 可实现往返播放）、`fill-mode`（`forwards` 让动画结束后**保持**最后一帧的状态，否则会跳回初始样式）、`play-state`（paused 可暂停）。

**animation 与 transition 的区别**是高频面试题，从三个维度对比：触发上，transition 必须有状态变化来触发（hover、改类名），animation 加上就能自动播放；能力上，transition 只能在两个状态间过渡，animation 支持多关键帧、循环、往复，表现力强得多；控制上，animation 可以用 animation-play-state 暂停恢复，并派发 animationstart / animationend / animationiteration 事件供 JS 监听，transition 只有 transitionend。选型经验：简单的状态切换反馈用 transition，复杂的多步动画、自动播放的加载动效用 animation。

## 十一、CSS 变量与进阶特性

### 1. 自定义属性 CSS 变量

CSS 变量（自定义属性）以 `--` 开头定义，用 `var()` 读取：在 `:root` 上定义即成为全局变量，`--primary: #409eff;` 配合 `color: var(--primary);` 使用。`var()` 支持第二个参数作为**兜底值**：`var(--gap, 8px)`，变量未定义时用 8px。

它的核心机制是**层叠继承**：变量像普通属性一样沿着 DOM 树继承，在任意元素上重新定义即可对该元素及其后代生效，这使"局部换肤"非常自然——在深色模式的根节点上重定义一组颜色变量，整个子树随之改变。变量是**运行时**生效的，JavaScript 可以通过 `element.style.setProperty("--primary", "red")` 动态修改、`getComputedStyle` 读取，主题切换、动态换肤基本都靠它实现。这也是它与预处理器变量（Sass 的 $var）的本质区别：后者在编译期被替换成死值，无法在运行时改变。

### 2. CSS3 新特性汇总

CSS3 按"模块"演进，新特性可以分几条线来记：**选择器**层面新增了属性选择器的部分匹配（^=、$=、*=）与结构伪类（nth-child 等）；**视觉**层面新增圆角（border-radius）、阴影（box-shadow、text-shadow）、渐变（linear/radial-gradient）、rgba/hsla 透明色；**动效**层面新增 transition、transform、animation；**布局**层面新增 flex 与 grid、多列布局；**响应式**层面新增媒体查询与视口单位；**其他**还有 @font-face 自定义字体、calc() 计算、自定义属性（CSS 变量）、overflow 滚动优化（-webkit-overflow-scrolling 等）。回答时按这几条线各举两例，比报菜名更能体现理解。

### 3. 伪类与伪元素

伪类和伪元素在 CSS2 时代都用单冒号书写，CSS3 开始为区分二者规定：**伪类用单冒号、伪元素用双冒号**（:hover 与 ::before），旧伪元素的单冒号写法仍被兼容。完整的清单与辨析见 [[#一、CSS 基础]]，这里补充实际使用中最高频的几个。

`:nth-child(n)` 系列是列表斑马纹和间隔样式的核心，公式 `an+b` 可以表达"从第 b 个开始每隔 a 个"：`:nth-child(3n+1)`。`:not()` 提高选择器表达力，如 `.item:not(:last-child)` 只给非最后一项加 margin。`:focus-within` 让"表单中任意控件聚焦时高亮整个容器"成为纯 CSS 能力。`::before` / `::after` 是伪元素的主力，必须设置 content 属性（哪怕是空字符串），默认是行内元素、不能设置宽高，需要时用 `display: inline-block` / `absolute` 转换；常用场景包括 clearfix、自定义图标、引号装饰、遮罩层。最后，`:has()` 作为"父选择器"补上了 CSS 表达能力的最后一块拼图，例如 `form:has(input:invalid)` 可以在表单包含非法输入时整体标红，此前必须靠 JS。

## 十二、CSS 常见面试问题

### 1. 水平垂直居中的实现方式

按推荐程度从上往下记。**flex 方案**：父元素 `display: flex; justify-content: center; align-items: center;`，两行搞定、不要求子元素尺寸已知，现代项目首选。**grid 方案**：父元素 `display: grid; place-items: center;`，更短，兼容性也早已成熟。

定位方案有两种，适用于父元素不便改布局的场景。其一是**absolute + transform**：子元素 `position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);`，先把自己左上角定位到中心，再向左上平移自身尺寸的一半，**不需要知道子元素宽高**；代价是 transform 创建层叠上下文、可能引发文字模糊，且会令 fixed 后代失效。其二是**absolute + margin: auto**：子元素 absolute 且 top/right/bottom/left 全为 0（或用 inset: 0），配 `margin: auto`，浏览器会把剩余空间均分给 auto 的 margin 实现居中，但它要求子元素有确定尺寸，且这类居中走的还是绝对定位，元素脱流。

传统方案里还有 `text-align: center` + `line-height` 等于容器高度（适合行内内容和已知高度的单行文字）、父元素 `display: table-cell; vertical-align: middle; text-align: center`（表格单元格特性，vertical-align 恰好在其上生效）等，作为补充了解即可。回答时先说 flex/grid，再以定位方案体现深度，最后提一句"绝对定位方案的对比点在于是否需要已知宽高"，层次就完整了。

### 2. 重排（回流）与重绘

**重排**（reflow，也叫回流）指元素的几何属性（尺寸、位置）发生变化，浏览器必须重新计算布局（重新生成布局树），波及范围可能是局部也可能是整棵树，**开销大**；**重绘**（repaint）指仅外观变化（颜色、背景、可见性），不改变几何，只需重新绘制像素，开销较小。两者关系是：触发重排必然重绘，重绘不一定重排。

常见触发源：重排来自增删 DOM、改变 width/height/margin/padding/display/position、读取 offsetTop / offsetWidth / getComputedStyle 等布局信息（浏览器被迫强制同步布局）；重绘来自 color、background、visibility、box-shadow 等外观属性。

减少的办法按思路分几类：**用合成层替代布局动画**——transform 和 opacity 的动画可以在合成层完成，不触发重排重绘，是性能优化的第一原则；**批量操作 DOM**——用文档片段一次性插入、把多次样式修改合并成切换一个类名；**读写分离**——把读布局的代码集中写完再统一写样式，避免读写交替造成的反复强制布局；**提前告知**——对频繁动画的元素用 will-change 提升到合成层；以及让频繁变化的元素脱离文档流（absolute/fixed），把重排影响限制在局部。

### 3. 隐藏元素的方式与区别

三种经典方式的核心差异用"是否占空间、能否交互、是否触发重排"三条线对比。`display: none` 完全不渲染：脱离文档流**不占空间**、不可交互、屏幕阅读器不读取；切换它会触发**重排**，代价最高；因为渲染树里没有它，也就**无法产生过渡效果**，子元素即使设回 display: block 也不会显示（display 不可被后代覆盖）。

`visibility: hidden` 隐藏但**占空间**，元素仍在渲染树中只是不绘制，仅触发重绘、开销较小；它是继承属性，子元素设置 `visibility: visible` 可以**单独显示**回来；配合 transition 有个实用技巧：过渡 visibility 可以实现"淡出后真正不可点击"的延迟隐藏。`opacity: 0` 透明度为零：**占空间**、仅触发重绘（通常直接在合成层处理，开销最小）、可以平滑过渡；但要特别注意**它仍然可以响应鼠标事件和键盘焦点**——视觉上看不见却能点到，如需不可交互要搭配 `pointer-events: none`。

其他方式作为补充：width/height 设为 0 配 overflow: hidden、`position: absolute` 移出视口、`clip-path` / `transform: scale(0)` 裁剪、`content-visibility` 跳过渲染。可访问性维度上：display: none 和 visibility: hidden 对屏幕阅读器不可见，而 opacity: 0 仍会被朗读，按需选择。

### 4. Flex 与 Grid 的区别与选型

最根本的区别是维度：**Flex 是一维布局**，一次只处理一行或一列（换行后行与行之间独立，不能直接控制"第 3 行第 2 格"）；**Grid 是二维布局**，行和列同时规划，天然适合整体页面骨架和规则的卡片矩阵。其次是驱动方式的差异：Flex 是**内容驱动**，项目尺寸由内容和 flex 属性协商，擅长"不知道内容多少"的组件内部排布（导航栏、工具条、按钮组、内容流）；Grid 是**布局驱动**，先用 grid-template 划定网格再把内容放进去，擅长"结构先定"的页面骨架。

选型经验法则：一维排布选 Flex，二维结构选 Grid，**二者混用是常态**——Grid 搭页面骨架（header / main / footer 区域），每个区域内部用 Flex 排内容。可以补充的加分点：Grid 有 grid-template-areas 命名区域让骨架一目了然，Flex 的对齐属性（justify/align）与 Grid 同名属性的含义恰好互换（Flex 的 justify 是主轴、Grid 的 justify 是水平轴），迁移时容易踩坑；至于瀑布流两者都做不到原生支持，用多列布局（column）实现。

### 5. CSS 选择器优先级如何计算（见 [[#一、CSS 基础]]）

---

上级：[[web前端开发/HTML+CSS/HTML+CSS|HTML+CSS]]

## 笔记

- [[web前端开发/HTML+CSS/八股文问题|八股文问题]]
- [[web前端开发/HTML+CSS/颜色|颜色]]
