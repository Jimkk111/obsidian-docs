---
title: HTML 精简版
created: 2026-09-08
updated: 2026-09-08
tags:
  - HTML+CSS
---

> [!info]
> 快速复习版，只保留结论和考点。详细讲解见 [[web前端开发/HTML+CSS/HTML|HTML]]。

## 一、HTML 基础

### 1. 文档结构与骨架

`<!DOCTYPE html>` 声明按 HTML 标准解析，**缺失会进入怪异模式**（盒模型等行为异常）。`<html lang="zh-CN">` 声明语言，供读屏器/翻译/SEO 使用。`<head>` 放元数据（不显示），`<body>` 放可见内容。

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>页面标题</title>
  </head>
  <body></body>
</html>
```

### 2. head 与元数据

- `<title>`：标签页标题、搜索结果标题、收藏夹默认名。
- `<meta charset="UTF-8">`：必须放在 head 前部，防乱码。
- `<meta name="viewport">`：移动端适配前提（见 [[#八、HTML 常见面试问题]]）；`<meta name="description">` 是搜索结果摘要。
- `<link>`：外部样式表、favicon、preload 预加载。
- `<script>` 三种加载：默认**下载+执行都阻塞解析**；`defer` 并行下载、DOM 解析完**按序**执行；`async` 并行下载、下完立即执行**不保序**。记忆：defer 保顺序等 DOM，async 下完就跑。

### 3. 元素、属性与语法

嵌套要合法：`<p>` 内不能放块级元素（浏览器会提前闭合）。**空元素**无结束标签：br、hr、img、input、meta、link、source、col。**布尔属性**存在即生效：disabled、checked、required、readonly。实体引用：`&lt; &gt; &amp; &quot; &nbsp;`。注释 `<!-- -->`。

## 二、文本内容

### 1. 标题与段落

每页一个 `<h1>`、**不跳级**、标题管层级不管样式。`<p>` 段落；`<br>` 只用于真正需要换行处（诗歌、地址），不拿来造间距；`<hr>` 表示话题转换。

### 2. 文本级语义

`<strong>`（重要，加粗）/ `<em>`（强调，倾斜）优于纯样式的 `<b>` / `<i>`。`<mark>` 高亮；`<pre>` 保留格式 + `<code>` 行内代码，常嵌套用。`<time datetime="...">` 机器可读时间；`<sub>` / `<sup>` 上下标；`<abbr title="全称">` 缩写；`<small>` 附注小字；`<del>` / `<ins>` 修订的删除与插入。

### 3. 列表

`<ul>` 无序、`<ol>` 有序（属性 start / reversed / type），内部只能直接放 `<li>`，嵌套列表写在 `<li>` 里。`<dl>` / `<dt>` / `<dd>` 描述列表，适合术语表、键值对。

### 4. 引用与联系信息

`<blockquote>` 块级引用（cite 属性给来源）；`<q>` 行内引用，浏览器**自动加引号**，别手写；`<cite>` 指作品名称；`<address>` 作者联系方式。

## 三、超链接

`href` 支持绝对/相对路径、`mailto:`、`tel:`；相对路径 `../` 上一级、`/` 开头从站点根出发。`href="#id"` 页内锚点跳转。外链新开页面用 `target="_blank"` 时**必须加 `rel="noopener noreferrer"`**（防新页面通过 window.opener 操控原页）。`download="文件名"` 触发下载，仅同源有效。链接文字要有意义（不要全用"点击这里"）。

## 四、图片与多媒体

### 1. 图片 img

`alt` 描述图片传达的**信息**；**装饰图必须写空 `alt=""`**（屏幕阅读器跳过），省略属性则会被朗读出路径。显式写 `width` / `height` 让浏览器预留空间，**防布局偏移（CLS）**。`loading="lazy"` 懒加载，首屏大图不要加。格式：照片 JPEG、透明 PNG、图标 SVG、追求体积 WebP/AVIF。

### 2. 响应式图片

**分辨率切换**用 `srcset`（候选图及固有宽度 `w`）+ `sizes`（这张图实际多宽），浏览器按视口宽度和 DPR 自选；**艺术指导**（不同断点不同裁剪/格式）用 `<picture>` + 多个 `<source>` + 兜底 `<img>`，也用于 WebP 降级。

### 3. audio 与 video

常用属性：controls、loop、muted、preload（none/metadata/auto）、poster（video 封面）。**autoplay 必须配 muted 才生效**（浏览器自动播放策略）。多个 `<source>` 按顺序兜底格式；`<track kind="subtitles" srclang="zh" src="x.vtt">` 加字幕。

### 4. 嵌入其他技术

`<iframe>` 是独立浏览上下文：内部 DOM/样式与父页隔离，通信靠 postMessage；嵌入不可信内容加 `sandbox`，视口外可 `loading="lazy"`。`<embed>` / `<object>` 用于浏览器渲染 PDF 等，object 可写降级内容。`<canvas>` 像素级 JS 绘制，适合高频重绘，缩放失真；`<svg>` 矢量、是 DOM 一部分可 CSS/JS 操作，适合图标。**静态可交互用 svg，高频重绘用 canvas**。

## 五、表格

`<tr>` 行，`<th>` 表头（配 `scope="col|row"`）+ `<td>` 数据格。语义分区 `<thead>` / `<tbody>` / `<tfoot>`，`<caption>` 是 table 的**第一个子元素**。合并单元格：`colspan` 跨列、`rowspan` 跨行，被合并的格子要从源码删掉。`<colgroup>` + `<col>` 按列设样式。表格只用于表格数据，不用来布局。

## 六、表单与控件

### 1. form 元素

`action` 提交地址；`method`：get 数据在 URL（搜索查询）、post 在请求体（登录、上传）；`enctype`：**文件上传必须 `multipart/form-data`**。提交过程：触发校验 → 收集所有**有 name 且未 disabled** 的控件成 `name=value` → 按 method 发送。所以：没写 name 的控件不会被提交；`disabled` 不提交，`readonly` 提交。form 之间不能嵌套。

### 2. input 类型与常用属性

类型：text、password、email、number（min/max/step）、checkbox、radio（**name 相同才互斥**）、file（multiple、accept）、date、color、range、hidden（不显示但提交）等。属性：placeholder 不能代替 label；required、pattern（正则）、minlength/maxlength；选对 type = 免费校验 + 对应键盘。

### 3. 其他表单控件

`<select>` + `<option>`（selected 默认选中、multiple 多选、optgroup 分组）。`<textarea>` 的初始值写在**标签内容里**，不是 value 属性。`<datalist>` + `<input list>` 提供可自由输入的建议。**`<button>` 在 form 内默认 `type="submit"`**，纯按钮必须显式写 `type="button"`；button 可包含 HTML 而 input 按钮只能纯文本。

### 4. label 与可访问性

点 label 等于点控件（扩大热区），也是读屏器朗读控件名称的来源。关联方式二选一：`<label for="id">` 显式（推荐），或把控件包进 label 隐式。

### 5. 客户端表单校验

约束属性：required、type、pattern、min/max、minlength/maxlength，提交时自动校验并阻止提交。伪类 `:valid` / `:invalid` / `:required` / `:placeholder-shown`。约束校验 API：`checkValidity()` 布尔校验、`reportValidity()` 校验并提示、`validity.valueMissing / patternMismatch / typeMismatch...` 定位错误、`setCustomValidity("msg")` 自定义错误（空串表示通过）。`<form novalidate>` 关闭内置校验。**客户端校验只是体验优化，服务端必须再校验**。

## 七、文档结构与语义化

### 1. 页面结构标签

```html
<header>页头</header>
<nav>导航</nav>
<main>页面唯一的主要内容</main>
<footer>页脚</footer>
```

`<section>` 主题区块（应有标题）、`<article>` 独立可分发内容（文章、评论、卡片）、`<aside>` 弱相关内容（侧边栏）。article 还是 section：问"放进 RSS 单独出现还有意义吗"。`<div>` / `<span>` 无语义，**找不到合适语义标签时的最后手段**。

### 2. 语义化的意义

用恰当语义的标签表达"这是什么"而非"长什么样"。好处：**SEO**（爬虫靠标签理解结构权重）、**可访问性**（读屏器按语义导航）、**可读性可维护性**、**环境兼容**（阅读模式等）。

## 八、HTML 常见面试问题

### 1. 行内元素、块级元素与行内块元素的区别

| | 块级 | 行内 | 行内块 |
|---|---|---|---|
| 换行 | 独占一行 | 不换行 | 不换行 |
| 宽高 | 可设置 | 无效 | 可设置 |
| 默认宽 | 撑满父元素 | 内容宽 | 内容宽 |
| 例子 | div、p、h1~h6、ul、li | a、span、strong、em | img、input、button、select |

`display: block / inline / inline-block` 互相转换。img 等可替换元素默认按 inline-block 方式处理。

### 2. HTML5 新增了哪些特性

语义化标签（header/nav/main/section/article/aside/footer）；多媒体 audio/video 取代 Flash；canvas 与内联 SVG；表单增强（新 type 与 required/pattern 等属性）；本地存储 localStorage / sessionStorage / IndexedDB；WebSocket、Web Worker、SSE；地理定位、拖放、History API；DOCTYPE 简化。

### 3. src 与 href 的区别

`href` 是**引用关联**（a、link）：建立文档与资源的关系，并行下载不阻塞解析。`src` 是**嵌入替换**（img、script、iframe）：资源成为文档的一部分；其中 `<script src>` 默认**阻塞解析**（可 defer/async 优化）。一句话：href 关联、src 嵌入；script 阻塞、link 不阻塞。

### 4. meta viewport 的作用

移动端默认用 980px 宽的布局视口排版再缩小，导致字很小。`width=device-width, initial-scale=1.0` 让布局视口等于设备理想宽度、初始缩放 1，**是 rem/vw/媒体查询等移动端方案的前提**。不建议 `user-scalable=no`（伤害可访问性，iOS 也会忽略）。

### 5. 对 HTML 语义化的理解（见 [[#七、文档结构与语义化]]）

---

上级：[[web前端开发/HTML+CSS/HTML+CSS|HTML+CSS]]

## 笔记

- [[web前端开发/HTML+CSS/HTML|HTML（详细版）]]
- [[web前端开发/HTML+CSS/八股文问题|八股文问题]]
- [[web前端开发/HTML+CSS/颜色|颜色]]
