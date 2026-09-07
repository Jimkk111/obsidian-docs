---
title: JavaScript
source: https://www.yuque.com/mook-mvqcp/wc9wsu/uh3kgt637ta0ek6o
created: 2026-05-16
updated: 2026-09-07
tags:
  - JavaScript
---

> [!info] 目录说明
> 目录结构参照《现代 JavaScript 教程》（[javascript.info](https://zh.javascript.info)）的三大部分体系，并参考 MDN Web Docs 与《JavaScript 高级程序设计（第 4 版）》补齐面试高频主题。标注「待补充」的小节为尚未填充内容，按需逐步完善。
> DOM / BOM 的详细笔记在同级目录：[[web前端开发/WebAPIs/WebAPIs|WebAPIs]]。

# 第一部分：JavaScript 编程语言

## 一、语言基础与环境

### 1. JavaScript 简介与规范（待补充）

语言历史、ECMAScript 与 JavaScript 的关系、官方手册与规范、MDN 的使用。

### 2. 开发者控制台与调试（待补充）

Chrome DevTools：Console、Sources、断点调试、debugger 语句。

### 3. 严格模式 "use strict"（待补充）

## 二、变量与数据类型

### 1. 变量声明：var、let、const 的区别（待补充）

### 2. JS 的数据类型有哪些，有什么区别（待补充）

原始类型（7 种）与引用类型的区别、存储方式（栈与堆）。

### 3. null 和 undefined

已被声明但未赋值的变量的值自动被 JavaScript 设置为 undefined，表示未定义状态。

null 是一个字面量，被用于有意的赋空值，表示一个空对象或空引用。

区别：undefined 的类型是 undefined，而 null 的类型是 object（历史遗留错误）；给函数的参数传参，传 undefined 会触发默认值，而传 null 不会，因为 null 是一个字面量。（这仅仅是区别的一部分，还有很多 null 和 undefined 的差异需要学习。）

### 4. 类型转换

字符串转数字：`parseInt()`

（待补充：Number()、隐式转换规则、其他类型间的转换）

### 5. 类型判断

- `typeof`：操作符，返回类型的字符串表示。
- `instanceof`：运算符，判断对象是否是某个构造函数的实例。

（待补充：Object.prototype.toString.call()、Array.isArray()、constructor）

### 6. 松散相等（==）与严格全等（===）

**讲解一下松散相等（==）与严格全等（===）是什么以及有什么区别**（待补充）

### 7. 判断空值的方法

**判断空值的方法有哪些**（待补充）

## 三、运算符与流程控制

### 1. 基础运算符与数学运算（待补充）

算术运算符、赋值运算符、自增/自减、逗号运算符、`+` 的类型转换行为。

### 2. 比较运算符（待补充）

### 3. 逻辑运算符与空值合并 ??（待补充）

`||`、`&&`、`!`、`??` 与 `||` 的区别。

### 4. 条件分支：if、三元运算符、switch（待补充）

### 5. 循环：while、for、for...of、for...in（待补充）

### 6. break、continue 与标签（待补充）

## 四、函数基础

### 1. 函数声明与函数表达式（待补充）

### 2. 箭头函数基础（待补充）

### 3. 参数：默认参数、剩余参数（待补充）

## 五、对象基础

### 1. 对象字面量与属性操作（待补充）

### 2. 对象引用与复制：浅拷贝与深拷贝（待补充）

**浅拷贝与深拷贝的实现方式有哪些**（待补充）

### 3. 垃圾回收（待补充）

### 4. 对象方法与 this（待补充）

### 5. 构造器与操作符 new（待补充）

### 6. 可选链 ?.（待补充）

### 7. Symbol 类型（待补充）

### 8. 对象转原始值（待补充）

## 六、内置数据类型详解

### 1. 数字类型 Number（待补充）

精度问题（0.1 + 0.2）、NaN 与 Infinity、Math 对象、Number 的方法。

### 2. 字符串 String

- `trim()`：去除字符串首尾的空白字符。
- `split(string)`：按指定分隔符把字符串拆分成数组（与数组的 join 互为逆操作）。
- 模板字符串（待补充）。
- 其他常用方法（待补充）：charAt、includes、indexOf、replace、slice、substring、toUpperCase / toLowerCase 等。

### 3. 数组 Array

- 截取：`slice([start], [end])`——截取数组中连续的一段元素浅拷贝。不指定开始和结束，表示浅拷贝所有元素；只指定一个元素表示开始位置，结束位置为末尾。字符串也有这个方法。
- 查找元素索引：`indexOf(searchValue, startIndex)` 从左往右查找第一个匹配的元素的索引；`lastIndexOf(searchValue, startIndex)` 从右往左查找。对于字符串也适用。
- 填充：`fill(el)`——用 el 填充数组的每一项。
- 拼接成字符串：`join(string)`——用指定分隔符把数组元素拼接成一个字符串。
- 增删改：push / pop / shift / unshift / splice（待补充）。
- 高频方法：**map、filter、reduce、forEach、find、some、every、flat 等方法的用法与区别**（待补充）→ 详见 [[web前端开发/JavaScript/高频使用的方法|高频使用的方法]]

### 4. 可迭代对象 Iterable（待补充）

### 5. Map 与 Set（待补充）

### 6. WeakMap 与 WeakSet（待补充）

### 7. Object.keys / values / entries（待补充）

### 8. 解构赋值（待补充）

### 9. 日期 Date（待补充）

### 10. JSON 方法与 toJSON（待补充）

## 七、函数进阶

### 1. 作用域与作用域链（待补充）

全局、函数、块级作用域；变量提升与暂时性死区。

### 2. 闭包（待补充）

概念、应用场景、内存泄漏问题。

### 3. 全局对象（待补充）

### 4. 递归与堆栈（待补充）

### 5. Rest 参数与 Spread 语法（待补充）

### 6. 函数对象与 new Function（待补充）

命名函数表达式（NFE）、自定义 call/apply 的实现。

### 7. 调度：setTimeout 与 setInterval（待补充）

### 8. 装饰器与转发：call、apply、bind（待补充）

三者的区别与手写实现。

### 9. 箭头函数与普通函数的区别（待补充）

## 八、对象属性配置

### 1. 属性标志与属性描述符（待补充）

### 2. 属性的 getter 和 setter（待补充）

## 九、原型与继承

### 1. 构造函数、原型、原型链（待补充）

**构造函数、原型（prototype）、原型链（\_\_proto\_\_）的机制**（待补充）

### 2. F.prototype 与原型继承（待补充）

### 3. 原生的原型（待补充）

### 4. 原型方法：Object.create、getPrototypeOf 等（待补充）

## 十、类与面向对象

### 1. Class 基本语法

类是原型链机制的语法糖，本质上一样，只不过增强了某些限制，并且是双链继承（构造函数继承和原型继承）。

类本身就是构造函数，它的 constructor 等价于构造函数内部的代码片段。

### 2. 静态属性和静态方法

静态属性和方法不在 constructor 里面定义，而是使用 static 关键字在类的顶层定义。

### 3. 属性与方法的分类

- 属性的分类：定义在类顶层和 constructor 内部的实例属性，静态属性。
- 方法的分类：定义在 constructor 里面的实例方法，定义在类顶层的原型方法，静态方法。

### 4. 私有属性和方法

私有属性必须定义在类顶层，并且以 # 开头。

### 5. 继承的实现方式有哪些（待补充）

原型链继承、借用构造函数继承、组合继承、寄生组合继承、class extends。

### 6. 类检查：instanceof（待补充）

### 7. Mixin 模式（待补充）

## 十一、错误处理

### 1. try...catch...finally 与 throw（待补充）

### 2. 自定义 Error，扩展 Error（待补充）

## 十二、异步编程

### 1. 同步与异步、回调函数（待补充）

回调地狱问题。

### 2. Promise：状态与 then / catch / finally（待补充）

三种状态、链式调用。

### 3. Promise 链与错误处理（待补充）

### 4. Promise API：all、allSettled、race、any（待补充）

### 5. 微任务 Microtask（待补充）

### 6. async / await（待补充）

### 7. 事件循环：宏任务与微任务（待补充）

**浏览器的事件循环机制**（待补充）

## 十三、生成器与高级迭代

### 1. Generator 生成器（待补充）

### 2. 异步迭代和 generator（待补充）

## 十四、模块

### 1. script 标签详解

script 标签的作用是在 HTML 文档中嵌入可执行的 JavaScript 代码，有 type、src、defer、async 等属性。

**type 属性**表示 script 标签内代码的类型，常见取值：

- `text/javascript`：默认值
- `application/json`
- `application/babel`
- `module`：表示 JavaScript 模块。可以使用 import 和 export，即支持模块语法；作用域隔离，不用担心全局作用域污染问题；默认延迟执行（defer 行为）；通过 src 引入外部模块时，服务器必须配置有效的 `Access-Control-Allow-Origin` 响应头；自动处理依赖。

**src 属性**指定 JavaScript 脚本来源。添加了 src 属性的 script 标签的内联代码将被忽略。

defer 与 async 的区别见 [[#二十一、脚本加载与页面生命周期]]。

### 2. 模块简介与导出导入 import / export（待补充）

### 3. 动态导入 import()（待补充）

## 十五、Proxy 与 Reflect

### 1. Proxy（待补充）

### 2. Reflect（待补充）

## 十六、正则表达式

正则表达式用于定义模板来匹配符合条件的字符串。

### 1. 基本匹配

`.` 指代任意一个字符；`*` 指代任意多个字符。

### 2. 范围匹配

有时候一个正则表达式可能包含多种字符串模板，这时候就需要用到 `|` 符号，表示或。

一个正则表达式包含的字符串模板太多，而这些字符串模板可能只是某个字符有差异而已，这时可以用 `[]` 来表示这个字符的取值范围。`[]` 里面可以是具体的字符，也可以用 `-` 指代一个字符集合，比如 `A-Z` 表示 26 个大写字母。在集合的前面写一个 `^` 表示取反。

### 3. 转义符号

如果需要在正则表达式里面使用元字符，则必须通过 `\\` 转义。

### 4. 重复元字符（量词）

如果一个字符的匹配次数不确定或者匹配次数很多，那么可以使用重复元字符简化正则表达式的书写。

- `?` 表示匹配 0 次或 1 次
- `*` 表示匹配 0 次或多次
- `+` 表示匹配 1 次或多次
- `{n}` 表示匹配 n 次
- `{n,}` 表示至少匹配 n 次
- `{n,m}` 表示匹配次数在 n 和 m 之间

（待补充：贪婪量词和惰性量词）

### 5. 定位元字符（锚点）

- `^` 表示前缀
- `$` 表示后缀

（待补充：词边界 \b、多行模式 m）

### 6. 字符类（待补充）

`\d`、`\w`、`\s` 及其大写取反形式、Unicode 修饰符 u。

### 7. 捕获组与反向引用（待补充）

### 8. 前瞻断言与后瞻断言（待补充）

### 9. 修饰符 flags：g、i、m、s、u、y（待补充）

### 10. 正则与字符串的方法（待补充）

match、matchAll、replace、search、split、test、exec。

## 十七、其他常用高级主题

### 1. 防抖与节流（待补充）

### 2. 柯里化（待补充）

### 3. BigInt（待补充）

---

# 第二部分：浏览器：文档、事件、接口

> 详细笔记在同级目录：[[web前端开发/WebAPIs/WebAPIs|WebAPIs]]（[[web前端开发/WebAPIs/DOM API|DOM API]]、[[web前端开发/WebAPIs/BOM APIs|BOM APIs]]）

## 十八、浏览器环境与 DOM

### 1. 浏览器环境概览：window、DOM、BOM（待补充）

### 2. DOM 树与节点类型（待补充）

### 3. 遍历与查找：getElement\*、querySelector\*（待补充）

### 4. 节点属性：type、tag、content，特性与属性（待补充）

### 5. 修改文档：创建、插入、移除节点（待补充）

### 6. 样式和类：class、style、getComputedStyle（待补充）

### 7. 元素尺寸、滚动与坐标（待补充）

→ 展开见 [[web前端开发/WebAPIs/DOM API|DOM API]]

## 十九、事件

### 1. 设置事件的方式

#### （1）DOM0 事件

#### （2）addEventListener('event', callback, useCapture)

useCapture 控制该监听器的执行时机，默认为 false（冒泡阶段执行）；与 stopPropagation 不同，后者是直接停止了该事件的冒泡，即阻断了所有元素的监听器执行。

### 2. 常见事件

**（1）鼠标事件**

- dblclick：鼠标双击事件。
- click、mousedown、mouseup、mousemove 等（待补充）。

**（2）键盘事件**（待补充）

keydown、keypress、keyup。

**（3）表单事件**

- input：用户输入事件。
- change：表单元素值改变事件。
- submit：表单提交事件。
- focus：元素获取焦点事件。
- blur：元素失去焦点事件。

**（4）其他事件**

- load：页面或资源加载完成事件。
- scroll：窗口或元素滚动事件。
- resize：窗口尺寸变化事件。
- contextmenu：右键菜单事件。
- paste：粘贴事件。
- copy：复制事件。
- cut：剪切事件。

### 3. 事件对象

事件对象 event 在事件流动过程中其 target 不会改变，但是 currentTarget（绑定了事件监听的元素）会改变。

事件流动其实指的就是 event 的流动。

（待补充：preventDefault、stopPropagation、stopImmediatePropagation）

### 4. 事件流与事件冒泡

事件冒泡是被广泛采用的一个事件流模型，包含三个阶段：事件捕获、到达目标（触发事件的元素，即 e.target 指向的元素）、事件冒泡。

### 5. 事件委托

有多个子元素共用同一个事件处理逻辑时，可以将事件监听设置到父元素，借助事件冒泡机制通过 e.target 获取事件目标对象。

### 6. 浏览器默认行为与 preventDefault（待补充）

### 7. 创建自定义事件 CustomEvent（待补充）

### 8. UI 事件进阶（待补充）

鼠标移动 mouseover/out 与 mouseenter/leave 的区别、鼠标拖放、指针事件、滚动。

## 二十、表单与控件

### 1. 表单属性与方法（待补充）

### 2. 聚焦：focus 与 blur（待补充）

### 3. 事件：change、input、cut、copy、paste（待补充）

### 4. 表单提交与校验（待补充）

## 二十一、脚本加载与页面生命周期

### 1. script 的 defer 与 async（待补充）

两者的加载与执行时机对比。

### 2. 页面生命周期：DOMContentLoaded、load、beforeunload、unload（待补充）

### 3. 资源加载：onload、onerror（待补充）

## 二十二、BOM 与浏览器存储

### 1. window、location、history、navigator（待补充）

弹窗方法、跨窗口通信。

### 2. cookie 与 document.cookie（待补充）

### 3. localStorage 与 sessionStorage（待补充）

### 4. IndexedDB（待补充）

→ 展开见 [[web前端开发/WebAPIs/BOM APIs|BOM APIs]]

---

# 第三部分：网络与其他

## 二十三、网络请求

### 1. Fetch（待补充）

### 2. FormData 与文件上传（待补充）

### 3. 跨源请求与 CORS（待补充）

### 4. XMLHttpRequest（待补充）

### 5. WebSocket 与 Server-Sent Events（待补充）

### 6. URL 对象（待补充）

## 二十四、二进制数据与文件

### 1. ArrayBuffer 与类型化数组（待补充）

### 2. Blob、File 与 FileReader（待补充）

## 二十五、动画与 Web Components

### 1. JavaScript 动画与 requestAnimationFrame（待补充）

### 2. Web Components：Custom Elements 与 Shadow DOM（待补充）

---

上级：[[web前端开发/web前端开发|web前端开发]]

## 笔记

- [[web前端开发/JavaScript/高频使用的方法|高频使用的方法]]
- [[web前端开发/面试准备/八股/JavaScript八股文（标准版）|JavaScript 八股文（标准版）]]
- [[web前端开发/面试准备/八股/JavaScripts八股文（手写版）|JavaScript 八股文（手写版）]]
- [[web前端开发/WebAPIs/WebAPIs|WebAPIs]]
