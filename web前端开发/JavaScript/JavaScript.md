---
title: JavaScript
source: https://www.yuque.com/mook-mvqcp/wc9wsu/uh3kgt637ta0ek6o
created: 2026-05-16
updated: 2026-09-08
tags:
  - JavaScript
---

> [!info] 目录说明
> 目录结构参照《现代 JavaScript 教程》（[javascript.info](https://zh.javascript.info)）的三大部分体系，并参考 MDN Web Docs 与《JavaScript 高级程序设计（第 4 版）》补齐面试高频主题。
> DOM / BOM 的详细笔记在同级目录：[[web前端开发/WebAPIs/WebAPIs|WebAPIs]]。
> 手写实现类题目（深拷贝、call/apply/bind、防抖节流、Promise 等）展开在：[[web前端开发/面试准备/八股/JavaScripts八股文（手写版）|JavaScript 八股文（手写版）]]。

# 第一部分：JavaScript 编程语言

## 一、语言基础与环境

### 1. JavaScript 简介与规范

JavaScript 由 Brendan Eich 于 1995 年在 Netscape 用 10 天时间设计出来，最初叫 LiveScript，后借 Java 之名改名 JavaScript，但两者除了语法上有些表面相似外没有任何关系。

**ECMAScript 与 JavaScript 的关系**：ECMAScript（简称 ES）是语言规范，由 ECMA 国际组织的 TC39 委员会维护（规范文档 ECMA-262）；JavaScript 是对该规范的实现，除 ECMAScript 核心外还包括 DOM、BOM 等宿主环境提供的 API。Node.js 也是 ECMAScript 的一个实现，宿主环境换成的是 fs、http 等 Node API。

**TC39 流程**：新提案从 Stage 0（strawman）→ Stage 1（提案）→ Stage 2（草案）→ Stage 3（候选，浏览器可实验性实现）→ Stage 4（定稿，进入年度规范）。ES6（2015，后改名 ES2015）是史上最大一次更新，之后规范每年发布一版（ES2016、ES2017……）。

**官方手册与规范**：

- [MDN Web Docs](https://developer.mozilla.org/zh-CN/)：日常查阅首选，权威且更新及时。
- [现代 JavaScript 教程](https://zh.javascript.info)：中文系统学习首选。
- [ECMA-262 规范原文](https://tc39.es/ecma262/)：最精确但晦涩，用于确认边界行为。

**MDN 的使用**：搜索 `MDN + API/方法名`；重点看「语法」参数与返回值、「浏览器兼容性」表格、「规范」章节可跳到 ECMA-262 对应小节。

### 2. 开发者控制台与调试

**Console 面板**：

- `console.log / warn / error / info`：常规输出，error 会带堆栈。
- `console.table(arr | obj)`：以表格展示数组和对象。
- `console.dir(obj)`：以对象树形式展示（查看 DOM 元素的 JS 对象属性时很有用）。
- `console.time(label) / timeEnd(label)`：统计代码执行耗时。
- `console.trace()`：打印当前调用栈。
- 特殊用法：`$_` 上一次执行结果、`$0` Elements 面板中当前选中的元素。

**Sources 面板与断点调试**：

- 普通断点：点击行号设置；行号处还可右键添加 **条件断点**（表达式为 true 才暂停）和 **日志点**（不暂停只打印）。
- 暂停时可以：单步执行（F10 逐过程 / F11 逐语句）、查看 Call Stack（调用栈）、Scope（当前作用域变量）、Watch 添加监视表达式。
- **debugger 语句**：代码中写 `debugger;`，开启 DevTools 时执行到该行会自动暂停，相当于代码内置的断点。
- XHR/事件监听断点、Blackbox（黑盒化第三方脚本，调试时跳过其内部代码）。

### 3. 严格模式 "use strict"

ES5 引入的受限执行模式，用来开启更严格的语法检查和错误报告。

**启用方式**：`'use strict';` 必须位于整个脚本或函数体的**第一行**（脚本顶部全局生效，函数体第一行只对该函数生效）。

**主要差异**：

- 未声明的变量赋值直接报错（非严格模式下是隐式创建全局变量）。
- 非方法调用时 `this` 是 `undefined`（非严格模式指向全局对象）。
- 禁止 `with` 语句；禁止八进制字面量语法；禁止 `delete` 变量、函数名、函数参数。
- 函数参数名重复、给不可写属性赋值、给原始值设置属性都会报错（非严格模式静默失败）。
- `arguments` 不再与命名参数双向绑定。
- 保留字（implements、interface、let、static 等）不能作变量名。

**注意**：ES 模块（`type="module"`）和 class 内部代码默认就是严格模式，无需手动声明。

## 二、变量与数据类型

### 1. 变量声明：var、let、const 的区别

| 维度 | var | let | const |
| --- | --- | --- | --- |
| 作用域 | 函数作用域 | 块级作用域 | 块级作用域 |
| 变量提升 | 提升并初始化为 `undefined` | 提升但不初始化（TDZ） | 提升但不初始化（TDZ） |
| 重复声明 | 允许 | 报错 | 报错 |
| 重新赋值 | 允许 | 允许 | 不允许 |
| 挂到全局对象 | 是（`window.x`） | 否 | 否 |

- **暂时性死区（TDZ）**：`let/const` 声明的变量从块级作用域开始到声明语句之间不可访问，访问抛 ReferenceError——这就是“提升但未初始化”的含义。
- `const` 锁定的是**变量与值的绑定**，而不是值本身：`const` 声明的对象/数组，其属性和元素仍可修改；真正不可变需要 `Object.freeze()`。
- 经典面试题：`for (var i...)` + `setTimeout` 输出 10 个 10，而 `let` 输出 0~9。原因是 var 只有一个共享的函数作用域变量，let 每轮循环创建一个新绑定。
- 声明习惯：默认用 `const`，需要重新赋值才用 `let`，不再使用 `var`。

### 2. JS 的数据类型有哪些，有什么区别

JS 共有 8 种数据类型：**原始类型 7 种**——`string`、`number`、`boolean`、`undefined`、`null`、`symbol`（ES2015）、`bigint`（ES2020）；以及引用类型 **object**。

两者的区别：

- **存储方式**：原始类型的值直接存储在栈中，变量保存的是值本身，赋值是**值拷贝**；引用类型的实际数据存储在堆中，变量保存的是堆内存的地址（引用），赋值是**引用拷贝**，两个变量指向同一个对象，修改会互相影响。
- **不可变性**：原始值不可变，所有字符串方法都返回新字符串；对象可变。
- **比较方式**：原始类型按值比较（`'a' === 'a'`），引用类型按引用比较（`{} === {}` 为 false）。
- 原始类型没有属性和方法，但访问 `str.length` 时 JS 会临时包装成包装对象（String/Number/Boolean），用完立即销毁。

### 3. null 和 undefined

已被声明但未赋值的变量的值自动被 JavaScript 设置为 undefined，表示未定义状态。

null 是一个字面量，被用于有意的赋空值，表示一个空对象或空引用。

区别：undefined 的类型是 undefined，而 null 的类型是 object（历史遗留错误）；给函数的参数传参，传 undefined 会触发默认值，而传 null 不会，因为 null 是一个字面量。（这仅仅是区别的一部分，还有很多 null 和 undefined 的差异需要学习。）

### 4. 类型转换

**转字符串**：

- `String(value)`：null→"null"，undefined→"undefined"。
- `value.toString()`：null 和 undefined 没有该方法；`num.toString(radix)` 可指定进制。
- 模板字符串 `` `${value}` ``、`+ ''` 隐式转换。

**转数字**：

- `Number(value)`：整体转换，失败为 NaN。`null→0`、`undefined→NaN`、`''→0`、`'12px'→NaN`、`true→1`、`false→0`。
- `parseInt(str, radix)` / `parseFloat(str)`：从左往右逐字符解析，遇到非法字符停止，`parseInt('12px')→12`，开头空白被忽略。parseInt 永远忽略小数部分；建议始终传入 radix。
- 一元 `+`：等价于 `Number(value)`。
- 算术运算符（`- * / %`）会把操作数转成数字。

**转布尔**：

- `Boolean(value)`、`!!value`、逻辑上下文中隐式转换。
- **假值只有 6 个**：`''`、`0`（含 -0、0n）、`NaN`、`null`、`undefined`、`false`。其余（包括 `[]`、`{}`、`'0'`）都是真值。

**对象转原始值**：见 [[#8. 对象转原始值]]（ToPrimitive：valueOf → toString，或 Symbol.toPrimitive 优先）。

### 5. 类型判断

- `typeof`：操作符，返回类型的字符串表示。`typeof null === 'object'`（历史遗留 bug）；`typeof function(){} === 'function'`（函数是唯一 typeof 有特殊返回值的引用类型）；其余对象（数组、日期、正则）都返回 `'object'`，无法细分。
- `instanceof`：运算符，判断对象是否是某个构造函数的实例，原理是沿原型链查找 `constructor.prototype`。可以区分数组、日期等具体类型，但不能用于原始值。
- `Object.prototype.toString.call(x)`：最通用，返回内部类标签，如 `'[object Array]'`、`'[object Null]'`、`'[object Number]'`（装箱后的原始值也能判断）。
- `Array.isArray(x)`：判断数组的首选方法，能跨 iframe 正确工作（instanceof 在跨窗口场景会失效）。
- `x.constructor === Array`：简便但有缺陷——constructor 可以被改写，且 null/undefined 没有 constructor。

### 6. 松散相等（\==）与严格全等（=\==）

**讲解一下松散相等（\==）与严格全等（=\==）是什么以及有什么区别**

- `===`（严格相等）：先比较类型，类型不同直接返回 false；类型相同再比较值（引用类型比较地址）。**不做任何类型转换**。
- `==`（松散相等）：类型不同时会先做隐式类型转换再比较，规则大致为：
  - `null == undefined` 为 true，且它们与任何其他值都不相等；
  - `NaN` 与任何值（包括自身）都不相等；
  - 数字与字符串：字符串转数字后比较；
  - 布尔值参与比较：先转数字（true→1，false→0）；
  - 对象与原始值比较：对象先转原始值（ToPrimitive）再比较。
- **实践建议**：始终使用 `===` 和 `!==`；唯一例外是判断 null/undefined 可以写 `x == null` 同时覆盖两者。

### 7. 判断空值的方法

**判断空值的方法有哪些**

```js
// 1. 判断 null 或 undefined（同时覆盖两者）
if (x == null) { }

// 2. 判断 undefined（严格）
if (x === undefined) { }

// 3. 空字符串（防空白字符干扰先 trim）
if (typeof str === 'string' && str.trim().length === 0) { }

// 4. 空数组
if (Array.isArray(arr) && arr.length === 0) { }

// 5. 空对象（自身可枚举属性）
if (obj && typeof obj === 'object' && Object.keys(obj).length === 0) { }
// 更严格：连同原型上的可枚举属性一起判断
if (JSON.stringify(obj) === '{}') { }   // 粗略，值为 undefined 的属性会被丢掉

// 6. 通用「无意义值」判断：假值全部视为空
if (!value) { }   // null、undefined、''、0、NaN、false 都算空——注意 0 会被误判
```

注意 `JSON.stringify` 方法的缺陷：值为 `undefined`、函数、Symbol 的属性会被直接忽略，所以 `JSON.stringify({a: undefined})` 也是 `'{}'`。

## 三、运算符与流程控制

### 1. 基础运算符与数学运算

- 算术运算符：`+ - * / % **`（幂，ES2016）。`%` 是取余（不是取模），结果符号跟被除数。
- **`+` 的特殊性**：只要有一个操作数是字符串，就执行拼接而不是加法；其他算术运算符一律把操作数转成数字。`1 + '2' → '12'`，`'5' - 2 → 3`。
- 一元 `+`：`+str` 等价于 `Number(str)`，最短的转数字写法。
- 赋值运算符：`=` 以及复合赋值 `+= -= *= /= %= **=`、逻辑赋值 `??= ||= &&=`（ES2021）。
- 自增/自减：`++`/`--`，前置（先自增再使用）与后缀（先使用再自增）。
- 逗号运算符：`(a, b)` 依次求值，取最后一个值，多用于 for 循环中一次自增多个变量。
- 精度问题：`0.1 + 0.2 === 0.3` 为 false（IEEE 754 双精度浮点）。解决办法：`(0.1 + 0.2).toFixed(10)` 后比较、先乘 10 的幂转整数运算、或用 BigInt。

### 2. 比较运算符

- `> < >= <=`：两个都是字符串时按**字典序**（Unicode 码点）逐字符比较，`'apple' < 'banana'` 为 true；否则把操作数转成数字比较。
- 注意：`null` 转数字是 0，`undefined` 转数字是 NaN，所以 `null > 0` 为 false、`null == 0` 为 false、`null >= 0` 却为 true（`==` 与其他比较对 null 的处理规则不同）。
- `undefined` 参与任何大小比较结果都是 false（NaN 不与任何值比较成立）。

### 3. 逻辑运算符与空值合并 ??
`||`、`&&`、`!`、`??` 与 `||` 的区别。

- `||`：短路求值，返回**第一个真值**，全假返回最后一个值。
- `&&`：返回**第一个假值**，全真返回最后一个值；常用于「条件执行」（`cb && cb()`）。
- `!`：转布尔再取反；`!!value` 是转布尔的惯用法。
- `??`（空值合并，ES2020）：**只有左侧是 null 或 undefined 时才返回右侧**。

`??` 与 `||` 的区别：`||` 会把一切假值（0、''、false、NaN）都当作「空」跳过，而 `??` 只把 null/undefined 当作空。设置默认值时 `??` 更安全：

```js
const count = input ?? 0;      // input 为 0 时，count 是 0（正确）
const count2 = input || 0;     // input 为 0 时，count2 也是 0，但 ''/false 也会被替换
```

`??` 与 `&&`、`||` 混用必须加括号，否则语法错误。`??=` 只在左侧为 null/undefined 时赋值。

### 4. 条件分支：if、三元运算符、switch

- `if`：条件被隐式转为布尔（假值列表见 [[#4. 类型转换]]）。
- 三元运算符 `cond ? a : b`：是表达式有返回值，适合简单二选一赋值，嵌套过深可读性差。
- `switch`：用 `===` 严格比较 case 值（不会做类型转换）；命中后**贯穿执行**直到 break，利用这一点可以合并多个 case；`default` 不必放最后。
- switch 的比较是恒等，`case 1` 不会匹配 `'1'`。

### 5. 循环：while、for、for...of、for...in
- `while` / `do...while`：条件未知次数时使用，do...while 至少执行一次。
- `for (let i = 0; ...)`：需要索引/控制步长时使用。
- `for...of`：遍历**可迭代对象**（数组、字符串、Map、Set、NodeList 等），拿到的是**值**；不能直接遍历普通对象（对象默认不可迭代）；可以 `break/continue/return`（这一点比 forEach 强）。
- `for...in`：遍历对象的**字符串键**（含原型链上可枚举属性），拿到的是**键名**；遍历数组会连自定义属性和原型属性一起遍历，不推荐用于数组；配合 `hasOwnProperty` 过滤自身属性。
- `forEach/map` 等：见 [[web前端开发/JavaScript/高频使用的方法|高频使用的方法]]。

```js
const arr = ['a', 'b'];
for (const v of arr) console.log(v);      // a b（值）
for (const k in arr) console.log(k);      // '0' '1'（键，字符串）
for (const [k, v] of Object.entries(obj)) // 同时拿键值（普通对象推荐写法）
```

### 6. break、continue 与标签

- `break`：立即跳出整个循环（或 switch）；`continue`：跳过本次迭代进入下一轮。
- **标签（label）**：`outer: for (...) { for (...) { break outer; } }`，可以跳出多层嵌套循环；对 continue 则跳到指定循环的下一轮。标签只能用在循环或带 label 的语句块前。

## 四、函数基础

### 1. 函数声明与函数表达式

```js
// 函数声明：有提升，可在定义之前调用
function sum(a, b) { return a + b; }

// 函数表达式：赋值语句执行时才创建，之前调用报错
const sum = function (a, b) { return a + b; };
```

- 函数声明的提升优先于变量提升：同名时函数声明先提升。
- 函数表达式配合 IIFE（立即执行函数表达式）在 ES 模块出现前用来制造私有作用域：

```js
(function () { /* 私有作用域 */ })();
```

- 函数是「一等公民」：可以赋值给变量、作为参数（回调函数 callback）、作为返回值返回。

### 2. 箭头函数基础

```js
const add = (a, b) => a + b;          // 表达式体：隐式 return
const square = x => { return x * x; } // 语句体：需要大括号和 return
const getUser = () => ({ id: 1 });    // 返回对象字面量需要包一层括号
```

- 只有一个参数时可省略括号；无参数或多参数必须有括号。
- 没有自己的 `this`，使用**外层词法作用域的 this**（定义时确定，而非调用时）；也没有 `arguments`。
- 不能用 `new` 调用（没有 `[[Construct]]`），没有 `prototype` 属性，不能作 Generator。
- 适合：回调、需要沿用外层 this 的场景。不适合：对象方法、原型方法、需要动态 this 的场合（如事件处理器里需要当前元素时）。

### 3. 参数：默认参数、剩余参数

```js
// 默认参数：调用时缺省（传 undefined）才生效，传 null 不会触发
function f(x, y = x * 2) { return x + y; }
f(2);    // 6，默认值可以使用前面的参数
```

- 默认值也可以是表达式/函数调用，**每次调用时才求值**。
- 解构参数：`function f({ name = 'anon' } = {}) {}`。

```js
// 剩余参数：把多余实参收集成一个真数组，必须是最后一个参数
function sum(...nums) { return nums.reduce((s, n) => s + n, 0); }
sum(1, 2, 3); // 6
```

- `arguments` 是类数组（没有数组方法），剩余参数是真数组，现代代码一律用剩余参数替代 `arguments`。

## 五、对象基础

### 1. 对象字面量与属性操作

```js
const name = 'Tom';
const user = {
  name,                 // 属性简写
  ['key_' + 1]: 'v',    // 计算属性名
  sayHi() {}            // 方法简写（与 sayHi: function() {} 等价，但不是箭头函数）
};
```

- 两种访问方式：点 `obj.key`（标识符）与方括号 `obj['key']`（任意字符串/变量/Symbol），动态键只能用方括号。
- 属性存在检查：`'key' in obj`（含原型链）与 `obj.hasOwnProperty('key')`（仅自身，ES2022 推荐改用 `Object.hasOwn(obj, 'key')`）。
- 删除属性：`delete obj.key`。
- 遍历：`for...in`（键，含原型链可枚举）、`Object.keys/values/entries`（自身可枚举）。
- 对象的键会被自动转成字符串（Symbol 除外），数字键 `1` 和字符串键 `'1'` 是同一个属性。
- 属性顺序：整数类键按数值升序排列在最前，其余按键的创建顺序。

### 2. 对象引用与复制：浅拷贝与深拷贝
**浅拷贝与深拷贝的实现方式有哪些**

对象赋值只是复制引用。浅拷贝只复制第一层，嵌套对象仍共享引用；深拷贝递归复制所有层级。

**浅拷贝**：

```js
const copy1 = { ...obj };                     // 展开运算符
const copy2 = Object.assign({}, obj);
const copy3 = arr.slice();                    // 数组
const copy4 = arr.concat();                   // 数组
```

**深拷贝**：

```js
// 1. JSON 方案：简单但有硬伤
const a = JSON.parse(JSON.stringify(obj));
// 缺陷：undefined / 函数 / Symbol 属性丢失；Date 变字符串；RegExp/Map/Set 变 {}；
// NaN、Infinity 变 null；循环引用直接报错；丢失原型。

// 2. structuredClone（现代首选，ES2022 进入标准）
const b = structuredClone(obj);
// 支持：循环引用、Date、RegExp、Map、Set、ArrayBuffer；
// 不支持：函数、Symbol、DOM 节点、原型链（都变普通对象）。

// 3. 手写递归（面试常考）：用 WeakMap 记录已拷贝对象解决循环引用
function deepClone(target, map = new WeakMap()) {
  if (target === null || typeof target !== 'object') return target;
  if (target instanceof Date) return new Date(target);
  if (target instanceof RegExp) return new RegExp(target);
  if (map.has(target)) return map.get(target);
  const clone = Array.isArray(target) ? [] : {};
  map.set(target, clone);
  for (const key of Object.keys(target)) {
    clone[key] = deepClone(target[key], map);
  }
  return clone;
}
```

完整实现（含 Map/Set/原型保留）见 [[web前端开发/面试准备/八股/JavaScripts八股文（手写版）|JavaScript 八股文（手写版）]]。生产环境使用 lodash 的 `_.cloneDeep`。

### 3. 垃圾回收

JS 的内存管理是自动的：垃圾回收器（GC）周期性找出「不再使用」的内存并释放。

- **可达性（reachability）**：基本算法思想。从根（全局对象、当前调用栈中的变量等）出发，能通过引用链到达的对象就是「可达」的，不可达的对象会被回收。
- **引用计数**（早期 IE 的实现）：记录每个对象被引用的次数，为 0 就回收。缺陷是**无法处理循环引用**（a 引用 b、b 引用 a，即使都不可达计数仍不为 0，导致内存泄漏）。
- **标记清除（mark-and-sweep）**：现代引擎的主流方案。从根开始标记所有可达对象，然后清除未标记的对象，循环引用自然被解决。
- 常见内存泄漏场景：被遗忘的定时器、闭包持有大对象/DOM 引用、脱离 DOM 的元素仍被 JS 变量引用、事件监听未解绑。
- `WeakMap/WeakSet/WeakRef` 的键是弱引用，不阻止对象被 GC，专门用于这类场景。

### 4. 对象方法与 this
- 方法简写 `obj.sayHi()` 调用时，`this` 指向 `obj`。**this 的值在调用时确定**（谁调用指向谁），而不是定义时。
- 常见指向规则：
  - `obj.fn()` → obj；`fn()` 独立调用 → 严格模式 undefined / 非严格 window；
  - 箭头函数没有自己的 this，沿用定义处外层的 this，且无法被 call/apply/bind 改变；
  - `new Fn()` → 新创建的实例；
  - `fn.call/apply/bind(obj)` → 显式指定的对象；
  - DOM 事件处理函数 → 绑定监听的元素。
- **丢失 this**：把方法提取出来单独调用（`const f = obj.fn; f()`）或作为回调传入时，this 会丢失，解决办法是 bind 或包一层箭头函数。

### 5. 构造器与操作符 new

```js
function User(name) { this.name = name; }
const u = new User('Tom');
```

`new` 的执行过程（四步）：

1. 创建一个空的普通对象 `{}`；
2. 把它的 `[[Prototype]]` 指向构造函数的 `prototype`；
3. 以该对象为 this 执行构造函数体；
4. 构造函数若显式返回一个对象，则 `new` 的结果用它替换；否则返回第一步创建的对象。

`new User()` 无参调用可省略括号。`new.target` 在构造函数内判断是否通过 new 调用（可用它实现「不用 new 也能正确构造」）。本质上 class 只是这套机制的语法糖。

### 6. 可选链 ?.

ES2020 引入，安全地访问嵌套属性，前值为 null/undefined 时**短路**返回 undefined 而不报错。

```js
user?.address?.street        // 属性访问
user?.['address']            // 方括号访问
user.sayHi?.()               // 方法调用（方法不存在也不报错）
arr?.[0]                     // 数组项
```

- 只用于**读取**和删除（`delete obj?.key`），不能用于赋值左侧。
- `?.` 前面的变量必须已声明（它不检查未声明的变量，只检查 null/undefined）。

### 7. Symbol 类型

ES2015 引入的第 7 种原始类型，表示**全局唯一**的标识符，常用作对象的「隐藏」属性键。

```js
const id = Symbol('id');        // 描述仅用于调试，任何两个 Symbol 都不相等
const obj = { [id]: 123 };      // 必须用方括号作键
obj[id];                        // 123
```

- Symbol 键**不会被 for...in、Object.keys、JSON.stringify 枚举**（但会被 Object.assign 和展开运算符复制），适合存放「不想被序列化/遍历」的内部数据。
- **全局注册表**：`Symbol.for('key')` 按字符串全局查找/创建，同名返回同一个；`Symbol.keyFor(sym)` 反查。
- 内置 Symbol（well-known symbols）：`Symbol.iterator`（定义 for...of 行为）、`Symbol.toPrimitive`（定义对象转原始值）、`Symbol.hasInstance`（自定义 instanceof 行为）。

### 8. 对象转原始值

抽象操作 ToPrimitive，`==` 比较对象、算术运算等场景触发。

```js
obj[Symbol.toPrimitive] = function (hint) { /* hint: 'number' | 'string' | 'default' */ };
```

没有 Symbol.toPrimitive 时的规则：

- hint 为 `'number'`（如一元 `+`、数学运算）：先调 `valueOf()`，结果不是原始值再调 `toString()`；
- hint 为 `'string'`（如模板字符串、String(obj)）：先 `toString()` 再 `valueOf()`；
- hint 为 `'default'`（如 `+`、`==`）：按 number 顺序（先 valueOf），但 Date 特殊——先 toString。
- `{} + 1` 的坑：行首的 `{}` 会被当成语句块，实际是 `+1`；`({}) + 1` 才是 `"[object Object]1"`。

## 六、内置数据类型详解

### 1. 数字类型 Number

JS 只有 Number 一种数字类型：IEEE 754 **双精度 64 位浮点**，整数和浮点没有区分。

- **精度问题**：`0.1 + 0.2 !== 0.3`（二进制无法精确表示 0.1）。整数运算只要在安全范围内是精确的：`Number.MAX_SAFE_INTEGER = 2^53 - 1`，超出用 BigInt。
- **NaN**：Not a Number，任何数学运算得不到合法数字时的结果；`NaN !== NaN`；`isNaN('12px')` 为 true（会先转数字），`Number.isNaN` 不转换、只对真正的 NaN 返回 true，更安全。
- **Infinity / -Infinity**：除以 0 得到 Infinity（不是报错）。
- 字面量：`0b1010` 二进制、`0o17` 八进制、`0xff` 十六进制、`1_000_000` 数字分隔符、`1.2e6` 科学计数。
- 常用方法：`toFixed(n)`（四舍五入保留 n 位小数，返回字符串）、`toPrecision(n)`、`toString(radix)`（转进制）。
- 静态方法：`Number.isInteger / isFinite / isNaN / isSafeInteger`（不转换参数）、`Number.parseInt / parseFloat`（等价于全局函数）、`Number.EPSILON`（最小精度误差，用于浮点比较）。
- **Math 对象**：`Math.floor / ceil / round / trunc`、`Math.abs / max / min / pow / sqrt / sign`、`Math.random()`（[0, 1)，取整需自行处理）、`Math.round(-1.5)` 是 -1（.5 时向 +∞ 方向取整）。

### 2. 字符串 String

- `trim()`：去除字符串首尾的空白字符。
- `split(string)`：按指定分隔符把字符串拆分成数组（与数组的 join 互为逆操作）。
- **字符串不可变**：所有方法都返回新字符串，原字符串不变。
- **模板字符串**：反引号包裹，支持 `${表达式}` 内嵌任意表达式和函数调用；支持多行书写（保留换行）；可嵌套使用。
- 其他常用方法：

```js
str.charAt(0)        // 取索引处字符（越界返回 ''）；str[0] 更常用，越界返回 undefined
str.at(-1)           // 支持负索引（ES2022）
str.includes('a')    // 是否包含，返回布尔
str.indexOf('a')     // 首次出现索引，找不到返回 -1
str.startsWith('a') / endsWith('a')
str.replace(reg|str, newStr|fn)  // 字符串参数只替换第一个；replaceAll 全部替换
str.slice(start, end)  // 负索引从尾部算；substring 不支持负索引且参数会交换
str.toUpperCase() / toLowerCase()
str.repeat(n)        // 重复 n 次
str.padStart(n, '0') / padEnd(n)   // 补齐长度，常用于时间格式化 '05'
str.codePointAt(i)   // 取码点（正确处理代理对）
```

### 3. 数组 Array

- 截取：`slice([start], [end])`——截取数组中连续的一段元素浅拷贝。不指定开始和结束，表示浅拷贝所有元素；只指定一个元素表示开始位置，结束位置为末尾。字符串也有这个方法。
- 查找元素索引：`indexOf(searchValue, startIndex)` 从左往右查找第一个匹配的元素的索引；`lastIndexOf(searchValue, startIndex)` 从右往左查找。对于字符串也适用。
- 填充：`fill(el)`——用 el 填充数组的每一项。
- 拼接成字符串：`join(string)`——用指定分隔符把数组元素拼接成一个字符串。
- **增删改**：

```js
arr.push(item)      // 尾部添加，返回新长度
arr.pop()           // 尾部删除，返回被删元素
arr.unshift(item)   // 头部添加，返回新长度
arr.shift()         // 头部删除，返回被删元素
arr.splice(start, deleteCount, ...items)  // 万能增删改：从 start 删 deleteCount 个再插入 items，返回被删除元素组成的数组，原数组被修改
```

- **排序与反转**（都会修改原数组）：

```js
arr.sort((a, b) => a - b)   // 默认按字符串 Unicode 排序，10 会排在 2 前面，数字必须传比较器
arr.reverse()
arr.sort(() => 0.5 - Math.random())  // 不是真正的洗牌算法，仅示意
```

- **判断**：`Array.isArray(arr)`。
- **创建/转换**：`Array.from(类数组或可迭代, mapFn)`（常用 `[...arguments]` 替代或转换 NodeList）、`Array.of(1, 2, 3)`、`[...new Set(arr)]` 去重。
- 高频方法：**map、filter、reduce、forEach、find、some、every、flat 等方法的用法与区别** → 详见 [[web前端开发/JavaScript/高频使用的方法|高频使用的方法]]

### 4. 可迭代对象 Iterable

实现了 `Symbol.iterator` 方法的对象就是可迭代对象，可以被 `for...of`、展开运算符 `...`、解构、`Array.from`、`Promise.all` 等消费。

```js
const range = {
  from: 1, to: 3,
  [Symbol.iterator]() {
    let cur = this.from, last = this.to;
    return {                       // 迭代器对象
      next() {
        return cur <= last
          ? { done: false, value: cur++ }
          : { done: true };
      }
    };
  }
};
[...range];   // [1, 2, 3]
```

- 迭代器协议：`next()` 返回 `{ value, done }`；可迭代协议：`[Symbol.iterator]()` 返回一个迭代器。
- 内置可迭代：Array、String、Map、Set、TypedArray、NodeList、arguments。**普通对象不可迭代**（`for...of {}` 报错）。
- 展开运算符 `...iterable` 可以把任何可迭代对象展开成数组。

### 5. Map 与 Set

**Map**：键值对集合，**键可以是任意类型**（包括对象、NaN）。

```js
const map = new Map([['a', 1]]);
map.set('b', 2).set(obj, 'x');   // set 返回 map 本身，可链式
map.get('a'); map.has('b'); map.delete('a'); map.clear();
map.size;
for (const [k, v] of map) {}     // Map 默认可迭代，插入顺序遍历
map.keys(); map.values(); map.entries(); map.forEach((v, k) => {});
```

Map 与 Object 的对比：Object 的键只能是字符串/Symbol，Map 键任意；Map 有 size 且记住插入顺序；Map 在频繁增删键值对的场景性能更好；Object 可以通过原型访问到继承的键，Map 不会。需要「记录元数据到对象」时用 `WeakMap`。

**Set**：成员值唯一的集合。

```js
const set = new Set([1, 2, 2, 3]);   // {1, 2, 3}，自动去重
set.add(4); set.has(1); set.delete(1); set.size;
const unique = [...new Set(arr)];    // 数组去重惯用法
```

### 6. WeakMap 与 WeakSet

- 键/成员**必须是对象**；持有的引用是**弱引用**——不计入垃圾回收的引用计数，对象的其他引用都消失后可被 GC 回收。
- 不可遍历、没有 size/clear，因为任意时刻成员都可能被回收，无法保证遍历结果。
- 典型用途：给 DOM 节点或第三方对象附加额外数据而不造成内存泄漏；实现真正的私有属性。

```js
const cache = new WeakMap();
function process(obj) {
  if (!cache.has(obj)) cache.set(obj, heavyCompute(obj));
  return cache.get(obj);   // obj 被回收后，缓存自动清理
}
```

### 7. Object.keys / values / entries
```js
Object.keys(obj)       // 自身可枚举字符串键 -> ['a', 'b']
Object.values(obj)     // -> [1, 2]
Object.entries(obj)    // -> [['a', 1], ['b', 2]]
Object.fromEntries(entries)  // 逆操作，由键值对数组生成对象
```

- 只包含**自身的、可枚举的、字符串键**（Symbol 键用 `Object.getOwnPropertySymbols`，全部键用 `Reflect.ownKeys`）。
- 与 `for...in` 的区别：for...in 还会遍历**原型链**上的可枚举属性；Object.keys 只遍历自身。
- 惯用组合：`Object.entries(obj).map(([k, v]) => [k, v * 2])` 再 `fromEntries` 回去；`Object.entries(obj).length === 0` 判断空对象。

### 8. 解构赋值
```js
// 数组解构（按位置）
const [a, b = 2, ...rest] = [1, undefined, 3, 4];   // a=1, b=2, rest=[3,4]
let x = 1, y = 2; [x, y] = [y, x];                  // 交换变量

// 对象解构（按键名，可重命名 + 默认值）
const { name, age: userAge = 18, address: { city } = {} } = user;

// 函数参数解构：命名参数效果
function draw({ x = 0, y = 0, w = 100 } = {}) {}

// 嵌套与默认值组合
const { data: { list = [] } = {} } = res;
```

- 解构失败得到 undefined，右侧为 null/undefined 直接报错（它们不能被解构），所以要给参数整体设默认值 `= {}`。
- 字符串也可解构：`const [c1, c2] = 'hi'`。
- 解构用于跳过无用返回值：`const [, second] = arr`。

### 9. 日期 Date
```js
const now = new Date();                    // 当前时间
const d1 = new Date(2026, 8, 8);           // 年,月(0起!),日 —— 9月8日
const d2 = new Date(1700000000000);        // 时间戳（毫秒）
const d3 = new Date('2026-09-08');         // 字符串解析（格式不兼容浏览器，不推荐）
```

- 获取：`getFullYear / getMonth(0起) / getDate(日) / getDay(星期0-6) / getHours / getMinutes / getSeconds / getMilliseconds`。
- 设置：对应的 `setFullYear / setMonth / ...`。
- 时间戳：`d.getTime()`、`Date.now()`（当前时间戳，最常用）、`+new Date()`。
- 格式化输出：`d.toISOString()`（UTC ISO8601 字符串）、`d.toLocaleString('zh-CN')`、`d.toJSON()`。
- 月份和星期从 0 开始是最常见的坑；`new Date()` 内部值是 UTC 毫秒数，本地方法按本地时区显示。
- 实际项目推荐 dayjs（2KB，API 兼容 moment）处理日期。

### 10. JSON 方法与 toJSON

```js
JSON.stringify(obj, replacer, space)
JSON.parse(str, reviver)
```

**stringify 规则**：

- 值为 `undefined`、函数、Symbol 的**对象属性被忽略**；数组中则变成 `null`；作为顶层值直接返回 `undefined`。
- `Date` 调用 `toJSON()` 变成 ISO 字符串；`NaN/Infinity` 变 `null`；RegExp 变 `{}`；Map/Set 变 `{}`。
- 循环引用直接抛 TypeError。
- 第二个参数：数组（白名单属性）或函数 `(key, value) => ...` 转换器；第三个参数：缩进空格数，用于美化输出。
- 对象如果有 `toJSON()` 方法，stringify 时优先调用它。
- **parse 的 reviver**：`JSON.parse(str, (key, value) => key === 'date' ? new Date(value) : value)` 常用于还原日期。

JSON 不支持注释、不能保存 undefined/函数/循环引用，这是它与 JS 对象字面量的区别。

## 七、函数进阶

### 1. 作用域与作用域链
全局、函数、块级作用域；变量提升与暂时性死区。

- **词法作用域（静态作用域）**：作用域由代码**书写位置**决定，与调用位置无关；内层作用域可以访问外层作用域的变量。
- 三种作用域：全局作用域、函数作用域、块级作用域（let/const，ES6）。var 没有块级作用域，`if (true) { var x }` 的 x 是全局的。
- **变量提升（hoisting）**：var 声明提升并初始化为 undefined；函数声明整体提升（函数体也提升）；let/const 只提升声明、不初始化，声明前访问报 ReferenceError——这段区域叫**暂时性死区（TDZ）**。
- **作用域链**：每个执行上下文有一个变量环境（VO/词法环境），查找变量时沿 `[[OuterEnv]]` 指针逐层向外查找，直到全局；找不到则 ReferenceError。闭包就是函数连同这条链的引用被保留下来。

### 2. 闭包
概念、应用场景、内存泄漏问题。

**定义**：函数与其词法作用域的组合——内部函数引用了外部函数的变量，且内部函数在外部函数返回后仍可被调用，此时这些变量不会销毁。

```js
function createCounter() {
  let count = 0;                 // 被 innerFn 引用，形成闭包
  return function () { return ++count; };
}
const counter = createCounter();
counter(); // 1
counter(); // 2 —— count 对外不可见，实现了私有状态
```

**应用场景**：私有变量与模块模式、柯里化/偏函数、防抖节流、缓存（记忆化）、迭代器、循环中为每个元素保留独立状态（配合 let）。

**注意的问题**：

- 闭包引用的变量常驻内存，滥用可能造成内存泄漏；不再使用时置为 null 可释放。
- 闭包引用 DOM 元素而定时器/监听器未清理，会连带 DOM 无法回收。
- 经典题：`for (var i=0;i<3;i++) setTimeout(()=>console.log(i))` 输出 3 个 3（共享同一个 i）；用 IIFE 或 let 修复。

### 3. 全局对象

- 浏览器：`window`（同时也是窗口对象）；Node.js：`global`；跨环境标准：`globalThis`（ES2020）。
- 全局对象提供内置能力：`window.setTimeout`、`window.location` 等。
- **var 声明的全局变量和函数声明会挂到全局对象上**；`let / const / class` 不会（但仍在全局作用域中可访问）。
- 尽量减少全局变量：命名冲突、难以追踪、永不释放；现代实践用 ES 模块天然隔离作用域。

### 4. 递归与堆栈

- 每次函数调用都会在**调用栈**压入一个栈帧（参数、局部变量、返回地址），返回时弹出；嵌套过深触发 **栈溢出**（Maximum call stack size exceeded）。
- 递归的关键是**基线条件**（终止条件）和**递归条件**，先保证收敛。
- 典型应用：树的遍历（DOM 树、文件夹）、深拷贝、扁平化数组、阶乘/斐波那契。
- 优化：用**记忆化**缓存子问题结果（斐波那契从指数级降到线性）；尾递归理论上可被优化为循环，但 JS 引擎（V8）并未普遍实现，深递归可改用循环 + 显式栈。

### 5. Rest 参数与 Spread 语法

- **Rest**（收集）：`function f(...args)`、解构 `const [a, ...rest] = arr`、`const { x, ...others } = obj`。只能出现在最后。
- **Spread**（展开）：`f(...arr)` 传参、`[...arr1, ...arr2]` 合并数组、`{ ...defaults, ...opts }` 合并对象（浅拷贝）、`[...'hello']` 字符串转数组。
- 两者语法相同方向相反：rest 收集剩余，spread 展开逐个。
- `Math.max(...arr)` 是 spread 最常见的应用。

### 6. 函数对象与 new Function

函数也是对象：有 `name`（函数名）、`length`（**未指定默认值的形参个数**）属性，可以挂自定义属性。

**命名函数表达式（NFE）**：`const f = function fact(n) { return n <= 1 ? 1 : n * fact(n - 1); }`——`fact` 只在函数内部可见，用于递归，外部不可访问。

**new Function**：

```js
const sum = new Function('a', 'b', 'return a + b');   // 参数列表 + 函数体字符串
```

- 它创建的函数**作用域只有全局**，即使在外层函数中创建也访问不到外层局部变量（闭包失效）。
- 用途：运行时由字符串动态生成代码（如服务端下发的规则），属于特殊场景；配合严格性要求低的地方使用，需警惕注入风险。
- **自定义 call/apply**：把函数临时挂到目标对象上借 this 调用再删除——`obj[fn] = fn; obj[fn](...args); delete obj[fn]`，完整实现见 [[web前端开发/面试准备/八股/JavaScripts八股文（手写版）|JavaScript 八股文（手写版）]]。

### 7. 调度：setTimeout 与 setInterval

```js
const id = setTimeout(fn, delay, arg1, arg2);   // delay 毫秒，返回定时器 id
clearTimeout(id);
const id2 = setInterval(fn, interval);
clearInterval(id2);
```

- 回调不是「准点」执行：进入宏任务队列等待主线程空闲，实际延迟 ≥ 指定值；嵌套 setTimeout 超过 5 层后最小延迟被强制为 4ms。
- `setTimeout(fn, 0)` 也不是立即执行——下一个宏任务执行。
- 回调内的 this：非严格模式指向 window（严格模式 undefined），需要绑定或用箭头函数。
- **setInterval 的累积问题**：回调执行时间超过间隔会导致任务堆积、执行时机漂移；推荐用「链式 setTimeout」——回调末尾再 `setTimeout(自身, interval)`。
- 场景切换/组件卸载时务必 clearTimeout，防止定时器持有闭包造成泄漏和逻辑错乱。

### 8. 装饰器与转发：call、apply、bind

三者的区别与手写实现。

```js
fn.call(thisArg, a, b)        // 立即执行，参数逐个传
fn.apply(thisArg, [a, b])     // 立即执行，参数以数组传（伪数组也行）
const g = fn.bind(thisArg, a) // 不执行，返回永久绑定 this 的新函数；支持预置参数（偏函数）
```

- call 与 apply 唯一区别是参数形式；`fn.call(null)` 在非严格模式下把 this 绑到 window。
- bind 返回的**绑定函数**：this 被永久固定，再 call/apply 也改不了；用 new 调用绑定函数时 this 由 new 决定。
- 常见用途：`Array.prototype.slice.call(arguments)`（把类数组转数组，旧写法）、借用对象方法 `hasOwnProperty.call(obj, key)`、React 类组件回调 `this.xxx.bind(this)`。
- 手写实现思路与完整代码见 [[web前端开发/面试准备/八股/JavaScripts八股文（手写版）|JavaScript 八股文（手写版）]]。

### 9. 箭头函数与普通函数的区别

**讲解一下箭头函数与普通函数的区别**

| 维度 | 箭头函数 | 普通函数 |
| --- | --- | --- |
| this | 词法作用域的 this，定义时确定，不可改变 | 调用时确定，谁调用指向谁 |
| arguments | 没有 | 有（类数组） |
| new 调用 | 不能，报错 | 可以 |
| prototype | 没有 | 有 |
| generator | 不能用 `function*` | 可以 |
| 变量提升 | 表达式形式无提升 | 函数声明有提升 |
| 重复参数 | 不允许（严格模式） | 非严格模式允许 |

核心区别是 this：箭头函数捕获**定义时**所在外层作用域的 this，之后 call/apply/bind 都无法修改；因此箭头函数适合做回调，不适合做对象方法、原型方法和构造函数。

## 八、对象属性配置

### 1. 属性标志与属性描述符

对象的每个属性除了值之外还有三个**标志**：

- `writable`：能否修改值（false 时赋值静默失败，严格模式报错）。
- `enumerable`：能否被 for...in / Object.keys 枚举。
- `configurable`：能否删除该属性、能否修改标志（一旦设为 false 不能再改回 true，唯独 writable 可以从 true 改成 false）。

```js
// 查看
Object.getOwnPropertyDescriptor(obj, 'key');
Object.getOwnPropertyDescriptors(obj);
// 定义/修改
Object.defineProperty(obj, 'key', {
  value: 1, writable: false, enumerable: false, configurable: false
});
Object.defineProperties(obj, { a: {...}, b: {...} });
```

- 用 `Object.defineProperty` 只写部分标志时，未指定的默认为 false（普通赋值创建的属性标志全为 true）。
- 常用组合：`Object.freeze(obj)` = 全部 writable/configurable 为 false（浅冻结）；`Object.seal` = 禁止增删但可改值；`preventExtensions` = 禁止添加新属性。
- 访问器属性（getter/setter）的描述符是 `get / set`，与 `value / writable` 互斥。

### 2. 属性的 getter 和 setter

```js
const user = {
  firstName: 'Tom',
  lastName: 'Cat',
  get fullName() { return `${this.firstName} ${this.lastName}`; },
  set fullName(v) { [this.firstName, this.lastName] = v.split(' '); }
};
user.fullName;          // 像访问属性一样触发 getter
user.fullName = 'A B';  // 触发 setter
```

- 访问器属性没有值，本质是拦截读/写操作的函数，**从外部看仍是普通属性访问**。
- 用途：派生值、数据校验、兼容旧接口（把方法伪装成属性）、响应式框架的数据劫持基础（Vue2 用 defineProperty）。
- 在 class 中：`get price() {} / set price(v) {}` 定义在原型上。
- 注意 `Object.assign` 与展开运算符复制的是 getter 的**求值结果**，不会复制 setter。

## 九、原型与继承

### 1. 构造函数、原型、原型链
**构造函数、原型（prototype）、原型链（\_\_proto\_\_）的机制**

- 三个概念分清：
  - 构造函数 `Foo`：普通函数，配合 new 创建实例；
  - 显式原型 `Foo.prototype`：函数才有的属性，是个对象，内部含 `constructor` 指回 Foo；
  - 隐式原型 `obj.__proto__`（正式名称 `[[Prototype]]`，建议用 `Object.getPrototypeOf` 访问）：每个对象都有，指向其构造函数的 prototype。
- **原型链查找**：访问 `obj.x` 时，先查自身属性 → 没有则沿 `__proto__` 逐层向上查 → 直到 `Object.prototype` → 再往上 `Object.prototype.__proto__` 为 null，仍没找到返回 undefined。方法共享、继承都依赖这条链。
- 赋值 `obj.x = 1` 只写自身属性，不改原型；`in` 操作符会查原型链。

```js
function Foo() {}
const f = new Foo();
f.__proto__ === Foo.prototype;              // true
Foo.prototype.constructor === Foo;          // true
f.__proto__.__proto__ === Object.prototype; // true
Object.prototype.__proto__ === null;        // 原型链终点
```

### 2. F.prototype 与原型继承
- `new Foo()` 创建的实例的 `[[Prototype]]` 就是**当时**的 `Foo.prototype`；之后修改 `Foo.prototype` 对已有实例立即生效（引用同一对象），但**替换整个 prototype 对象**不影响已创建的实例（它们引用旧对象）。
- 默认 `Foo.prototype = { constructor: Foo }`；手动赋新对象时记得补 `constructor: Foo`，否则 `instanceof` 虽仍可用（走原型链），但 `f.constructor` 会丢失。
- 继承的读、写不对称：读走原型链，写永远写自身（除非是访问器）。
- 对象字面量 `{}` 的原型是 `Object.prototype`；`Object.create(null)` 创建无原型对象（纯净字典）。

### 3. 原生的原型

`String.prototype`、`Array.prototype`、`Object.prototype` 等内置对象的原型上挂着内置方法（`arr.push` 就来自 `Array.prototype`）。链路如：数组实例 → Array.prototype → Object.prototype → null。

- **不要修改内置原型**（污染全局、命名冲突），唯一例外是 polyfill（为旧浏览器补齐标准方法，且需检测是否已存在）。
- 可以用内置原型临时借用方法：`Array.prototype.join.call(arguments, '-')`。

### 4. 原型方法：Object.create、getPrototypeOf 等
```js
Object.create(proto, propsObj)   // 以 proto 为原型创建新对象，最纯粹的基于原型的继承
Object.getPrototypeOf(obj)       // 读取 [[Prototype]]（推荐）
Object.setPrototypeOf(obj, p)    // 设置 [[Prototype]]（性能差，只在创建时设置最好）
Object.hasOwn(obj, key)          // 自身属性检查（替代 hasOwnProperty）
obj.__proto__                    // getter/setter，仅浏览器兼容，已不推荐使用
```

- 读取原型应一律使用 `Object.getPrototypeOf`；设置原型尽量在 `Object.create` 创建时完成，避免 `setPrototypeOf`（V8 会使已优化的形状失效，性能明显下降）。
- `instanceof` 的原理：检查 `C.prototype` 是否出现在对象的原型链上，可用 `Symbol.hasInstance` 自定义。

## 十、类与面向对象

### 1. Class 基本语法

类是原型链机制的语法糖，本质上一样，只不过增强了某些限制，并且是双链继承（构造函数继承和原型继承）。

类本身就是构造函数，它的 constructor 等价于构造函数内部的代码片段。

```js
class User {
  // 类顶层声明的实例属性（ES2022）
  type = 'user';
  // 原型方法
  sayHi() {}
  // constructor：new 时执行，负责初始化实例属性
  constructor(name) {
    this.name = name;     // 实例属性
  }
}
typeof User;              // 'function'
User.prototype.sayHi;     // 方法都在 prototype 上
```

class 与构造函数的差异：class 声明不像函数声明那样提升；类内部强制严格模式；类的方法不可枚举；class 必须 new 调用；类内部所有方法没有 `[[Construct]]`（不能被 new）。

### 2. 静态属性和静态方法

静态属性和方法不在 constructor 里面定义，而是使用 static 关键字在类的顶层定义。

```js
class User {
  static version = '1.0';        // 静态属性
  static create(name) {          // 静态方法
    return new User(name);
  }
}
User.version;     // 通过类访问
User.create('Tom');
```

- 静态成员挂在**类本身**（`User.xxx`）上而非 `prototype`，实例访问不到。
- 静态方法中的 `this` 指向类本身；典型用途：工厂方法、工具函数（`Array.from`、`Object.create` 就是内置类的静态方法）。
- 静态继承：子类可以通过原型链访问父类的静态成员。

### 3. 属性与方法的分类

- 属性的分类：定义在类顶层和 constructor 内部的实例属性，静态属性。
- 方法的分类：定义在 constructor 里面的实例方法，定义在类顶层的原型方法，静态方法。

```js
class A {
  a = 1;              // 实例属性（每个实例独立）
  static s = 2;       // 静态属性（只有类本身有）
  m() {}              // 原型方法（所有实例共享）
  constructor() { this.b = 3; }   // 实例属性（constructor 内定义）
}
```

### 4. 私有属性和方法

私有属性必须定义在类顶层，并且以 # 开头。

```js
class Counter {
  #count = 0;             // 私有属性（真私有，外部无法访问）
  #increase() { return ++this.#count; }   // 私有方法
  get value() { return this.#count; }
}
new Counter().#count;     // SyntaxError（即使在类外用 typeof 探测也报错）
```

- `#` 私有是语言级强制隔离，与 TypeScript 的 `private`（仅编译期检查）不同。
- 私有属性只能在类内部访问；旧的「软私有」约定 `_prop` 只靠自觉。

### 5. 继承的实现方式有哪些
原型链继承、借用构造函数继承、组合继承、寄生组合继承、class extends。

```js
function Parent(name) { this.name = name; this.colors = ['red']; }
Parent.prototype.say = function () {};

// 1. 原型链继承：Child.prototype = new Parent()
// 缺点：引用类型属性被所有实例共享；无法向父构造函数传参
// 2. 借用构造函数：function Child() { Parent.call(this, name); }
// 优点：可传参、实例属性独立；缺点：不能继承原型上的方法
// 3. 组合继承 = 1 + 2：
function Child(name, age) {
  Parent.call(this, name);          // 第二次调用 Parent
  this.age = age;
}
Child.prototype = new Parent();     // 第一次调用 Parent（缺点：父构造函数执行两次）
Child.prototype.constructor = Child;

// 4. 寄生组合继承（ES5 最优解）：用 Object.create 代替 new Parent
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;

// 5. class extends：语法糖，内部即寄生组合思路
class Child extends Parent {
  constructor(name, age) {
    super(name);        // 必须先调用 super 才能使用 this
    this.age = age;
  }
}
```

ES6 的 `extends` 还能继承内置类型（`class MyArray extends Array`），静态方法也被继承。

### 6. 类检查：instanceof

- `obj instanceof Class`：检查 `Class.prototype` 是否在 obj 的**原型链**上（不检查 constructor 属性，改写 constructor 不影响结果）。
- 任何构造函数都可以用 `Symbol.hasInstance` 自定义 instanceof 行为。
- 判断原始值用 typeof；跨 iframe/frame 场景 instanceof 会失效（两个 window 的 Array.prototype 不同），改用 `Array.isArray`。
- `Object.prototype.toString.call()` 是最通用的终极方案。

### 7. Mixin 模式

不通过继承链、而是把多个对象的方法「混入」一个类，模拟多重继承。

```js
const sayMixin = {
  sayHi() { console.log(`Hello, ${this.name}`); }
};
class User {
  constructor(name) { this.name = name; }
}
Object.assign(User.prototype, sayMixin);   // 把 mixin 方法合并进原型

// 更结构化的写法：mixin 函数返回增强后的类
const Serializable = Base => class extends Base {
  toJSON() { return JSON.stringify(this); }
};
class User2 extends Serializable(User) {}
```

注意 mixin 不要覆盖目标类已有方法、不要引起循环引用；组合优于继承，mixin 是轻量复用的折中方案。

## 十一、错误处理

### 1. try...catch...finally 与 throw
```js
try {
  // 可能出错的代码（只捕获运行时错误，不捕获语法错误；只同步，不捕获异步回调里的错误）
  throw new Error('custom');   // throw 可抛任意值，但推荐 Error 对象（有 message/name/stack）
} catch (err) {
  // err.message / err.name / err.stack；不关心错误内容可省略绑定：catch {}
} finally {
  // 无论如何都会执行（即使 try/catch 中 return，也会先执行 finally）
}
```

要点：

- `catch` 会捕获 try 块内**所有**错误，粒度过粗时用 `instanceof`/`err.name` 区分处理，或缩小 try 范围。
- `finally` 中的 return 会**吞掉** try/catch 的返回值和异常，避免在 finally 里 return。
- 异步代码中的异常必须在 Promise 链（.catch）或 async 函数内的 try/catch 中捕获，普通的 try/catch 抓不到定时器回调里的错误。
- 浏览器兜底：`window.onerror` 与 `unhandledrejection` 事件用于上报未捕获错误。

### 2. 自定义 Error，扩展 Error
```js
class ValidationError extends Error {
  constructor(message) {
    super(message);          // 传入父类，设置 message
    this.name = 'ValidationError';
  }
}
function test() { throw new ValidationError('参数不合法'); }
try { test(); }
catch (err) {
  if (err instanceof ValidationError) { /* 业务处理 */ }
  else { throw err; }        // 不认识的错误继续向上抛（catch 后重新抛出）
}
```

- 必须调 `super(message)`，因为 Error 构造函数负责设置 message 和 stack。
- 在 Promise 链中，错误会沿链向下传递到最近的 catch，利用自定义 Error 类型实现「不同错误不同分支」。

## 十二、异步编程

### 1. 同步与异步、回调函数
回调地狱问题。

- **同步**：代码顺序执行，后一行等待前一行完成；阻塞耗时操作（网络请求、定时器）会卡住主线程（渲染也卡）。
- **异步**：发起后不等待，继续执行后续代码，结果就绪后通过**回调**通知。异步不会阻塞，但代码被拆散、顺序不再线性。
- **回调函数（callback）**：作为参数传给异步 API、在将来某时刻被调用的函数。
- **回调地狱（callback hell）**：多个异步任务存在依赖时，回调里再嵌回调，代码呈金字塔形，难以阅读、错误处理分散、无法方便地并行/取消——这正是 Promise 出现的原因。

```js
// 回调风格（Node 早期 error-first 约定）
fs.readFile('a.txt', (err, data) => {
  if (err) return handleError(err);
  fs.readFile('b.txt', (err, data2) => {
    if (err) return handleError(err);
    // 金字塔继续……
  });
});
```

### 2. Promise：状态与 then / catch / finally
三种状态、链式调用。

```js
const p = new Promise((resolve, reject) => {
  // executor 同步立即执行
  setTimeout(() => resolve('ok'), 1000);   // resolve(value) => fulfilled
  // reject(new Error('fail'))            // => rejected
});
```

- 三种状态：`pending → fulfilled` 或 `pending → rejected`，**一旦改变不可逆**。resolve 后再 resolve/reject 无效。
- `then(onFulfilled, onRejected)`：注册回调，**返回一个新 Promise**（不是原 Promise），这是链式调用的基础。
- `catch(onRejected)` 等价于 `then(null, onRejected)`，语义更清晰。
- `finally(fn)`：无论成功失败都执行 fn（不接受参数），返回的 Promise 透传之前的结果，常用于收尾（关闭 loading）。
- 状态之外，Promise 还区分 handled/unhandled：rejected 且无人 catch 会触发 `unhandledrejection`。

### 3. Promise 链与错误处理
```js
fetch(url)
  .then(res => res.json())        // 返回值传给下一个 then；返回 Promise 则等它 settle
  .then(data => process(data))
  .catch(handleError)             // 捕获链上任何一环的错误（ rejection 冒泡）
  .finally(cleanup);
```

- 每个 then 的返回值决定下一环：返回普通值 → 下一环 fulfilled；返回 Promise → 下一环等它 settle；抛错 → 下一环 rejected。
- **错误沿链冒泡**到最近的 onRejected/catch；catch 之后链恢复正常，可以继续 then。
- then 的第二个参数与 catch 的区别：catch 能捕获**它之前所有环节**的错误，包括上一个 then 的成功回调里抛的错；第二个参数只捕获自己前面一环。
- catch 放中间可以做「降级」：请求失败返回默认值，让后续环节继续。

### 4. Promise API：all、allSettled、race、any
静态方法都接收一个**可迭代**（通常是 Promise 数组），返回新 Promise：

| 方法 | 语义 | 任一失败时 | 全部失败时 |
| --- | --- | --- | --- |
| `Promise.all` | 全部成功才成功，结果按**传入顺序**组成数组 | 立即 rejected（第一个失败的原因） | rejected |
| `Promise.allSettled` | 等**全部结束**，返回 `[{status:'fulfilled',value}, {status:'rejected',reason}]` | 不影响其他 | 照常返回全部结果 |
| `Promise.race` | 第一个 settle 的结果（成功或失败） | 同左 | 同左 |
| `Promise.any` | 第一个**成功**的结果，忽略失败 | 继续等下一个成功 | AggregateError（含 errors 数组） |

- 并发场景：页面同时请求多个接口用 `all`（强一致，一损俱损）或 `allSettled`（容错，各自处理）；超时控制用 `race`（Promise.race([fetch, timeout(3000)])）。
- 传入非 Promise 值会被 `Promise.resolve` 包装。

### 5. 微任务 Microtask
- Promise 回调（then/catch/finally）不会立即执行，而是进入**微任务队列（microtask queue）**，在**当前同步代码执行完之后、下一个宏任务之前**统一清空执行。
- `queueMicrotask(fn)` 可手动入队微任务。
- 微任务里再产生的微任务会在同一轮清空过程中继续处理（队列清空到空为止），而宏任务要等下一轮。
- 这就是「Promise.then 先于 setTimeout 执行」的原因；也是 async/await 中 await 之后的代码表现为「同步续接」的机制。

### 6. async / await
```js
async function load() {
  try {
    const res = await fetch(url);        // await 只能在 async 函数或模块顶层
    const data = await res.json();
    return data;                         // async 函数的返回值自动包装成 Promise
  } catch (err) {                        // await 的 rejection 用 try/catch 捕获
    console.error(err);
  }
}
```

- `async` 函数**总是返回 Promise**；返回非 Promise 值会被包装。
- `await promise`：暂停该函数，把后续代码注册为微任务，等 Promise settle 后恢复；**不阻塞主线程**，只阻塞这个函数。
- await 非 Promise 值：直接以该值为结果（包装 resolve）。
- **串行 vs 并行**：`const a = await f1(); const b = await f2();` 是串行（总耗时相加）；先 `const pa = f1(); const pb = f2();` 再 await 是并行，或用 `Promise.all`。
- 循环中的 await：`for...of + await` 串行执行；需要并发时先 map 出 Promise 数组再 `Promise.all`。
- 顶层 await：ES 模块中可直接使用（CommonJS 不行）。

### 7. 事件循环：宏任务与微任务
**浏览器的事件循环机制**

- JS 是**单线程**的（同一时刻只执行一段代码），通过事件循环（Event Loop）实现异步调度。
- **宏任务（macrotask / task）**：整体 script 代码、setTimeout / setInterval 回调、I/O、UI 事件回调、postMessage。
- **微任务（microtask）**：Promise 的 then/catch/finally 回调、`queueMicrotask`、`MutationObserver`、await 之后的续接代码。
- 一轮循环的顺序：**执行一个宏任务 → 清空全部微任务队列 → （必要时执行渲染：rAF、style、layout、paint）→ 取下一个宏任务**。微任务具有最高优先级，且在每一轮中被彻底清空。

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// 输出：1 4 3 2  —— 同步 → 微任务 → 宏任务
```

- 补充：`requestAnimationFrame` 不属于宏任务也不属于微任务，它在渲染前执行；`setImmediate`（Node）与 `process.nextTick`（Node，优先级高于 Promise 微任务）是 Node 特有。

## 十三、生成器与高级迭代

### 1. Generator 生成器

`function*` 声明，函数体可以 `yield` 暂停和恢复执行。

```js
function* gen() {
  const x = yield 1;      // 暂停并产出 1；恢复时 next 的参数成为 yield 的值
  yield x + 10;
}
const it = gen();          // 调用不执行，返回迭代器
it.next();                 // { value: 1, done: false }
it.next(5);                // { value: 15, done: false }（x = 5）
it.next();                 // { value: undefined, done: true }
```

- 生成器对象既是迭代器又是可迭代对象，可直接 `for...of`（循环到 done 为止）和展开。
- `yield*` 委托给另一个生成器，组合复用。
- 用途：惰性生成无限序列、遍历树、实现自定义可迭代结构；曾经用 generator + 执行器（co 库）写异步流程，后被 async/await 取代。

### 2. 异步迭代和 generator

- **异步生成器**：`async function*`，内部可同时使用 `await` 和 `yield`，`next()` 返回 Promise。
- **异步可迭代协议**：`Symbol.asyncIterator`；消费方式是 `for await...of`（每轮等待 value 的 Promise settle）。

```js
async function* fetchPages(urls) {
  for (const url of urls) yield await fetch(url).then(r => r.json());
}
for await (const data of fetchPages(urls)) { console.log(data); }
```

- 展开运算符 `...` 需要**同步**迭代器，不能用于异步可迭代对象。

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

### 2. 模块简介与导出导入 import / export
```js
// export：具名导出（可多个）
export const PI = 3.14;
export function sum(a, b) {}
export default class App {}      // 默认导出（每个模块最多一个）

// import
import App from './app.js';              // 默认导出：名字任意
import { PI, sum as add } from './utils.js';  // 具名导出：花括号 + 可重命名
import * as utils from './utils.js';     // 命名空间导入
import './polyfill.js';                  // 仅执行副作用
```

- 模块规则：顶层 this 是 undefined；自动严格模式；模块只被**执行一次**（单例缓存）；import 声明被提升，导入路径在编译期静态分析（这也是 Tree-shaking 的基础）。
- 浏览器原生模块要求路径完整（`./` 开头，不含扩展名会 404），需 `type="module"`。
- CommonJS（Node，`require/module.exports`）是运行时动态导入、值拷贝；ESM 是静态的、**值的实时绑定**。

### 3. 动态导入 import()
```js
const module = await import('./module.js');   // 返回 Promise，resolve 模块命名空间对象
module.someExport;                            // 默认导出在 .default 上
import('./a.js').then(m => m.run());          // 非顶层也可用
```

- 类似函数调用但**不是函数**，不能用变量存别名 `const p = import` 调用。
- 参数可以是运行时计算的字符串（如拼接路径），实现**按需加载**；构建工具基于它做**代码分割（code splitting）**和路由懒加载。
- 静态 import 与动态 import() 的区别：静态在编译期解析、提升；动态在执行时解析、可出现在任何位置、返回 Promise。

## 十五、Proxy 与 Reflect

### 1. Proxy
Proxy 用于**包装目标对象**，拦截对它的基本操作（读取、赋值、函数调用等），实现代理。

```js
const proxy = new Proxy(target, {
  get(target, prop, receiver) {
    return prop in target ? target[prop] : 0;   // 默认值
  },
  set(target, prop, value, receiver) {
    if (typeof value !== 'number') throw new TypeError('必须是数字');
    return Reflect.set(target, prop, value);    // 交给默认行为
  },
  has(target, prop) {},          // 拦截 in
  deleteProperty(target, prop) {},
  apply(target, thisArg, args) {},   // 拦截函数调用
  construct(target, args) {}         // 拦截 new
});
```

- 常用 trap：`get`、`set`、`has`、`deleteProperty`、`ownKeys`、`apply`、`construct`。
- 典型应用：数据验证、默认值、负数组索引、日志/性能监控、**响应式系统**（Vue3 的 reactivity 用 Proxy 替代 Vue2 的 defineProperty，可监听新增/删除属性和数组下标）。
- 限制：内部槽依赖 this 的对象（Map/Set/Date）直接代理会导致方法报错，需要绑定 this 处理；Proxy 与原始 target 的 `===` 不相等，容易踩坑。

### 2. Reflect
Reflect 是一个静态方法集合，与 Object 上的方法对应，是「操作对象的默认行为」的标准化入口。

- 方法与 Proxy 的 trap 一一对应（get/set/has/deleteProperty/defineProperty/getOwnPropertyDescriptor/ownKeys/apply/construct 等）。
- 相比 Object 的优势：以**布尔返回值**报告操作成败（`Reflect.set` 返回 false，而 `Object.defineProperty` 失败直接抛错）；`Reflect.apply/target` 参数形式更合理。
- 与 Proxy 配合：在 trap 中用 `Reflect.xxx` 调用默认行为并正确转发 receiver，保证拦截后行为与原生一致。

## 十六、正则表达式

正则表达式用于定义模板来匹配符合条件的字符串。

创建方式：字面量 `/pattern/flags`（推荐，编译期解析）与 `new RegExp('pattern', 'flags')`（pattern 运行时才知道时使用）。

### 1. 基本匹配

`.` 指代任意一个字符；`*` 指代任意多个字符。

### 2. 范围匹配

有时候一个正则表达式可能包含多种字符串模板，这时候就需要用到 `|` 符号，表示或。

一个正则表达式包含的字符串模板太多，而这些字符串模板可能只是某个字符有差异而已，这时可以用 `[]` 来表示这个字符的取值范围。`[]` 里面可以是具体的字符，也可以用 `-` 指代一个字符集合，比如 `A-Z` 表示 26 个大写字母。在集合的前面写一个 `^` 表示取反。

### 3. 转义符号

如果需要在正则表达式里面使用元字符，则必须通过 `\\` 转义。

需要转义的元字符：`. * + ? ( ) [ ] { } ^ $ | \`。

### 4. 重复元字符（量词）

如果一个字符的匹配次数不确定或者匹配次数很多，那么可以使用重复元字符简化正则表达式的书写。

- `?` 表示匹配 0 次或 1 次
- `*` 表示匹配 0 次或多次
- `+` 表示匹配 1 次或多次
- `{n}` 表示匹配 n 次
- `{n,}` 表示至少匹配 n 次
- `{n,m}` 表示匹配次数在 n 和 m 之间

**贪婪与惰性**：量词默认**贪婪**（尽可能多地匹配），`'<b>text</b>'.match(/<.+>/)` 会匹配整个串。在量词后加 `?` 变成**惰性**（尽可能少地匹配）：`*?`、`+?`、`??`、`{n,m}?`，如 `/<.+?>/` 只匹配 `<b>`。

### 5. 定位元字符（锚点）

- `^` 表示前缀
- `$` 表示后缀

- 词边界 `\b`：匹配「单词字符与非单词字符的位置」而非字符本身，如 `\bjava\b` 不会匹配 `javascript`；`\B` 相反。
- 多行模式 `m`：加上后 `^` 和 `$` 匹配每一行的行首/行尾而不只是整个字符串的首尾。

### 6. 字符类

`\d`（数字 [0-9]）、`\w`（字母数字下划线 [A-Za-z0-9_]）、`\s`（空白：空格、\t、\n 等）；大写形式 `\D \W \S` 表示取反。

- `.` 等价于 `[^\n]`，任意非换行字符；s 修饰符（dotAll）让 `.` 也能匹配换行。
- Unicode 修饰符 `u`：正确处理码点大于 0xFFFF 的字符（如 emoji、生僻汉字），`/\p{Script=Han}/u` 可用 Unicode 属性类匹配。

### 7. 捕获组与反向引用
```js
// 捕获组：() 把子模式匹配的内容捕获，供后续引用
const [, year, month, day] = '2026-09-08'.match(/(\d{4})-(\d{2})-(\d{2})/);

// 命名捕获组：?<name>
const { yy, mm } = '2026-09'.match(/(?<yy>\d{4})-(?<mm>\d{2})/).groups;

// 非捕获组：(?: ) 只分组不捕获
/(?:ab)+/.exec('ababab');

// 反向引用：\1 引用第 1 个捕获组（正则内部）；替换时用 $1
'aaa bbb'.replace(/(\w+) (\w+)/, '$2 $1');   // 'bbb aaa'
```

### 8. 前瞻断言与后瞻断言
断言只匹配「位置」，不消费字符：

- `x(?=y)` 正向前瞻：x 后面必须紧跟 y（y 不算进结果）。
- `x(?!y)` 负向前瞻：x 后面不能是 y。
- `(?<=y)x` 正向后瞻（ES2018）：x 前面必须是 y。
- `(?<!y)x` 负向后瞻：x 前面不能是 y。

```js
'1px 2em'.match(/\d+(?=px)/g);          // ['1'] —— 只要后面是 px 的数字
'100$ 200元'.match(/\d+(?!元)/g);        // ['100', '20'] —— 排除后面紧跟元的数字
'价格 ¥25'.match(/(?<=¥)\d+/g);          // ['25'] —— 前面是 ¥ 的数字
```

### 9. 修饰符 flags：g、i、m、s、u、y

| 修饰符 | 含义 |
| --- | --- |
| `g` | global，全局匹配（exec 循环、match 返回全部、replace 全替换） |
| `i` | ignore case，忽略大小写 |
| `m` | multiline，多行模式，^ $ 匹配每行 |
| `s` | dotAll，`.` 匹配换行符 |
| `u` | unicode，完整 Unicode 支持 |
| `y` | sticky，粘性匹配，只能从 `lastIndex` 位置开始匹配 |

- 注意副作用：带 `g` 或 `y` 的正则**有状态**（lastIndex），在 test/exec 之间复用同一个正则对象会产生诡异结果。
- `regex.flags` 查看修饰符，`regex.source` 查看模式文本。

### 10. 正则与字符串的方法
match、matchAll、replace、search、split、test、exec。

```js
str.match(re)      // 有 g：返回所有匹配的数组（无捕获组信息）；无 g：返回首个匹配 + 捕获组
str.matchAll(re)   // 要求有 g，返回迭代器，每项含捕获组信息
str.replace(re|str, newStr|fn)   // $& 整个匹配、$1 捕获组、fn(m, p1, ..., offset, str) 自定义
str.search(re)     // 返回首个匹配索引，没有返回 -1
str.split(re)      // 按正则切分：'a,b;;c'.split(/[,;]+/)
re.test(str)       // 返回布尔，验证格式最常用
re.exec(str)       // 无 g 等价 match 无 g；有 g 时维护 lastIndex，配合循环取全部匹配
```

## 十七、其他常用高级主题

### 1. 防抖与节流
两者都是控制高频事件回调执行频率的手段。

**防抖（debounce）**：事件停止触发 n 毫秒后才执行一次；期间再次触发则重新计时。适合「最终态」场景：搜索输入联想、resize 结束后重算布局、表单校验。

```js
function debounce(fn, delay) {
  let timer = null;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

**节流（throttle）**：n 毫秒内最多执行一次（固定频率稀释触发）。适合「过程态」场景：scroll/鼠标移动、按钮防连点。

```js
function throttle(fn, interval) {
  let last = 0;
  return function (...args) {
    const now = Date.now();
    if (now - last >= interval) {
      last = now;
      fn.apply(this, args);
    }
  };
}
```

区别记忆：防抖是「最后一次说了算」，节流是「按固定节奏来」。带 leading/trailing 选项的完整实现见 [[web前端开发/面试准备/八股/JavaScripts八股文（手写版）|JavaScript 八股文（手写版）]]。

### 2. 柯里化
柯里化（currying）：把 `f(a, b, c)` 转换成 `f(a)(b)(c)` 的技术，每次传入部分参数返回新函数，参数收齐才执行。

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn.apply(this, args);
    return (...rest) => curried.apply(this, [...args, ...rest]);
  };
}
const add = curry((a, b, c) => a + b + c);
add(1)(2)(3);   // 6
add(1, 2)(3);   // 6
```

- `fn.length` 是形参个数，作为「参数收齐」的判断依据。
- 用途：**参数复用**（`const log = curry(base => level => msg => ...)` 固定前缀生成专用函数）、函数组合、延迟执行。
- 相关概念：偏函数（partial application）——固定部分参数，bind 的预置参数就是偏函数，柯里化是更彻底的偏函数。

### 3. BigInt
ES2020 引入的第 8 种原始类型，表示**任意精度整数**，突破 Number 的 2^53 安全整数上限。

```js
const big = 9007199254740993n;       // 字面量加 n
const big2 = BigInt('9007199254740993');
big + 1n;      // 与 BigInt 运算
big == 9007199254740993;   // true（== 允许混比，宽相等）
big === 9007199254740993;  // false（类型不同）
```

- 不能与 Number 直接做 `+ - * /` 等数学运算（报错），需先统一类型；除法向零取整：`5n / 2n → 2n`。
- 不能用于 Math 对象方法；不能用一元 `+`；`typeof` 返回 `'bigint'`。
- `JSON.stringify` 不支持（抛 TypeError）；`BigInt.asIntN / asUintN` 限制位数。
- 适用场景：大整数 ID（雪花算法）、加密运算、精确金融整数计算。

---

# 第二部分：浏览器：文档、事件、接口

> 详细笔记在同级目录：[[web前端开发/WebAPIs/WebAPIs|WebAPIs]]（[[web前端开发/WebAPIs/DOM API|DOM API]]、[[web前端开发/WebAPIs/BOM APIs|BOM APIs]]）

## 十八、浏览器环境与 DOM

### 1. 浏览器环境概览：window、DOM、BOM

JavaScript 在浏览器中的宿主环境由三部分组成：

- **window**：全局对象，同时是浏览器窗口的接口；全局变量、setTimeout 等都挂在其上。
- **DOM（Document Object Model）**：文档对象模型，把 HTML 文档解析成树，提供操作页面内容结构的 API。
- **BOM（Browser Object Model）**：浏览器对象模型，操作浏览器窗口本身：location、history、navigator、screen、弹窗等。

### 2. DOM 树与节点类型

HTML 被解析成 DOM 树，每个节点都是 Node 的子类实例，通过 `nodeType` 区分：

- 元素节点 `Node.ELEMENT_NODE = 1`、文本节点 3、注释节点 8、文档节点 9（document）、文档类型节点 10。
- `node.nodeName`（元素节点为大写标签名）与 `node.nodeValue`（文本/注释节点才有内容）。
- 元素是节点的子集：`element.children` 只含元素，`element.childNodes` 含文本和注释节点（含空白换行）。

### 3. 遍历与查找：getElement*、querySelector*

- 老式选择器：`getElementById`、`getElementsByClassName` / `getElementsByTagName`（返回**动态** HTMLCollection）。
- 现代选择器：`querySelector(css)` 返回第一个匹配元素、`querySelectorAll(css)` 返回**静态** NodeList（可用 forEach、可展开）。一律用 CSS 选择器语法，推荐统一使用。
- 匹配与包含：`el.matches(css)` 判断是否匹配选择器（事件委托常用）、`el.closest(css)` 向上找最近匹配祖先。
- 祖先/兄弟：`parentElement`、`children`、`firstElementChild`、`previousElementSibling`（Element 版都跳过文本节点）。

### 4. 节点属性：type、tag、content，特性与属性

- `nodeType`、`tagName`（仅元素）。
- 内容：`innerHTML`（含标签的 HTML，注意 XSS）、`textContent`（纯文本，性能好且安全）、`outerHTML`。
- **特性（attribute）与属性（property）**：attribute 是写在 HTML 标签上的（`el.getAttribute('data-x')`，值总是字符串）；property 是 DOM 对象的 JS 属性（`el.value`，类型多样）。大部分标准 attribute 与同名 property 双向同步，但 value 等例外；`dataset` 映射 `data-*` 特性。

### 5. 修改文档：创建、插入、移除节点

```js
const div = document.createElement('div');   // 创建
div.textContent = 'hello';
parent.append(el, 'text');   // 末尾插入多个（Node/字符串）；prepend 头部
el.before(x); el.after(x); el.remove();      // 相对插入与移除（现代 API）
parent.insertBefore(newEl, refEl);           // 传统 API
// 移动 = 插入已有节点（自动从原位置摘除）
el.insertAdjacentHTML('beforeend', html);    // 更快的批量 HTML 插入
```

- 批量插入大量节点时使用 `documentFragment` 或一次性 innerHTML，减少回流。

### 6. 样式和类：class、style、getComputedStyle

- 类操作：`el.classList.add/remove/toggle/contains`，优先用 class 而非逐条改样式。
- 内联样式：`el.style.width = '100px'`（驼峰命名，只读写内联样式）。
- 计算样式：`getComputedStyle(el).fontSize`——最终生效值（含样式表），只读；读取常用的 `offsetWidth/clientWidth` 等几何属性见下一节。

### 7. 元素尺寸、滚动与坐标

- 布局尺寸：`offsetWidth/Height`（含边框）、`clientWidth/Height`（内容 + padding）、`scrollWidth/Height`（内容总高，判断可滚动）。
- 滚动：`scrollTop/scrollLeft` 读写、`scrollTo/scrollBy({behavior:'smooth'})`、`scrollIntoView()`。
- 坐标：相对视口 `getBoundingClientRect()`、相对文档 `pageX/pageY`；`elementFromPoint(x, y)` 取指定坐标处的元素。

→ 展开见 [[web前端开发/WebAPIs/DOM API|DOM API]]

## 十九、事件

### 1. 设置事件的方式

#### （1）DOM0 事件

```js
el.onclick = function (e) {};   // 赋值，同一事件只能有一个处理函数
el.onclick = null;              // 解绑
```

#### （2）addEventListener('event', callback, useCapture)

```js
el.addEventListener('click', fn, { capture: false, once: false, passive: false });
el.removeEventListener('click', fn);   // 必须传入同一个函数引用
```

useCapture 控制该监听器的执行时机，默认为 false（冒泡阶段执行）；与 stopPropagation 不同，后者是直接停止了该事件的冒泡，即阻断了所有元素的监听器执行。

- `once: true` 执行一次后自动移除；`passive: true` 声明不调用 preventDefault（scroll 性能优化）。
- 匿名函数无法解绑；DOM0 与 DOM2 可共存，执行按注册顺序。

### 2. 常见事件

**（1）鼠标事件**

- dblclick：鼠标双击事件。
- click、mousedown、mouseup：按下与抬起；click 只有 mousedown 与 mouseup 发生在同一元素上才会触发。
- mousemove：移动；mouseover/mouseout 与 mouseenter/mouseleave 的区别见 [[#8. UI 事件进阶]]。
- wheel：滚轮事件。

**（2）键盘事件**

keydown、keypress、keyup。

- 触发顺序：keydown → （keypress）→ keyup；长按 keydown 连续重复触发。
- `e.key` 是可打印字符（'a'、'Enter'、'Escape'），`e.code` 是物理键位（'KeyA'），做快捷键用 e.key。
- keypress 已废弃，只对可打印字符触发；现代一律 keydown。

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

- `e.target`：实际触发事件的最深元素；`e.currentTarget`：当前正在执行监听器的元素（等于 this）。
- **preventDefault()**：阻止浏览器默认行为（链接跳转、表单提交、右键菜单），不影响事件传播。
- **stopPropagation()**：阻止事件继续传播（不再向上冒泡/向下捕获），同元素上的其他监听器仍执行。
- **stopImmediatePropagation()**：阻止传播**并且**阻止同元素上后续监听器执行。

### 4. 事件流与事件冒泡

事件冒泡是被广泛采用的一个事件流模型，包含三个阶段：事件捕获、到达目标（触发事件的元素，即 e.target 指向的元素）、事件冒泡。

- 捕获阶段：从 window → document → …… → 目标父级（只在 addEventListener 第三个参数为 true 时有监听器执行）。
- 目标阶段：e.target 处触发；冒泡阶段：目标 → 父级 → …… → window（默认监听在此阶段执行）。
- 个别事件不冒泡：focus/blur（对应的 focusin/focusout 冒泡）、load、mouseenter/mouseleave。

### 5. 事件委托

有多个子元素共用同一个事件处理逻辑时，可以将事件监听设置到父元素，借助事件冒泡机制通过 e.target 获取事件目标对象。

```js
ul.addEventListener('click', (e) => {
  const li = e.target.closest('li');
  if (!li || !ul.contains(li)) return;
  console.log(li.dataset.id);
});
```

优点：节省监听器数量、动态新增子元素无需重新绑定。注意：中间用 `e.preventDefault` 需谨慎；依赖自定义事件的场景需要在监听器里显式设置 `bubbles: true`。

### 6. 浏览器默认行为与 preventDefault

浏览器对很多事件有内置行为：a 标签跳转、表单提交刷新页面、右键菜单、文本拖拽、链接拖拽打开等。

```js
form.addEventListener('submit', (e) => { e.preventDefault(); });  // 手动接管提交
document.addEventListener('contextmenu', (e) => e.preventDefault());
```

- preventDefault 不阻止冒泡；stopPropagation 不阻止默认行为——两者独立。
- 判断事件是否可取消：`e.cancelable`；取消后可用 `e.defaultPrevented` 查询。

### 7. 创建自定义事件 CustomEvent
```js
const ev = new CustomEvent('my-event', {
  bubbles: true,           // 默认不冒泡
  detail: { userId: 1 }    // 附加数据，监听器中用 e.detail 读取
});
el.dispatchEvent(ev);
el.addEventListener('my-event', (e) => console.log(e.detail));
```

- 用普通 `new Event(type)` 只能传事件类型；CustomEvent 的价值在 `detail` 携带数据。
- 用于解耦模块间的通信（发布-订阅）；跨组件场景现代更多用事件总线/状态管理。

### 8. UI 事件进阶
鼠标移动 mouseover/out 与 mouseenter/leave 的区别、鼠标拖放、指针事件、滚动。

- **mouseover/mouseout 会冒泡**，进出子元素也会触发（e.relatedTarget 是来源/去向元素）；**mouseenter/mouseleave 不冒泡**、进入子元素不触发，适合「进入/离开整个区域」语义。
- 拖放（HTML5 DnD）：draggable="true" + dragstart / dragover（必须 preventDefault 才允许放置）/ drop / dragend。
- 指针事件（Pointer Events）：pointerdown/move/up 统一鼠标、触摸、笔；`pointerId` 支持多点；配合 `setPointerCapture` 实现可靠拖拽。移动端 touch 事件（touchstart/touchmove/touchend）逐渐被指针事件取代。
- 滚动：window 与元素都可触发 scroll（不冒泡但捕获可监听），配合 rAF 节流；惯性滚动监听 passive: true 提升性能。

## 二十、表单与控件

### 1. 表单属性与方法

```js
form.elements.name          // 按控件 name 访问（elements 集合）
form.method / action / enctype
form.reset()                // 重置为初始值
form.submit()               // JS 提交（不触发 submit 事件与校验）
input.disabled / readonly / required
input.value                 // 文本、select 的选中值
checkbox.checked
select.options / selectedIndex / multiple
```

### 2. 聚焦：focus 与 blur

- `el.focus()` / `el.blur()` 编程式聚焦；`autofocus` 属性页面加载即聚焦。
- 事件：focus/blur 不冒泡；focusin/focusout 冒泡（可委托）。
- `document.activeElement` 当前聚焦元素；`el.hasFocus()`（document 上）。
- Tab 键切换顺序：`tabindex` 属性控制（正数按序、0 默认、-1 不可 Tab 聚焦），影响可访问性需谨慎。

### 3. 事件：change、input、cut、copy、paste
- **input**：每次输入实时触发（打字、粘贴、拖拽都算），做实时搜索/字数统计用它。
- **change**：值改变**且失去焦点**（或 select/checkbox 切换）才触发，一次编辑只触发一次。
- **cut/copy/paste**：剪切板事件，可用 `e.clipboardData.getData('text')` 读写，常用于粘贴内容清洗（`e.preventDefault()` 后手动插入）。
- 键盘逐字符触发 keydown → input → keyup；中文输入法过程触发 compositionstart/update/end，此时 input 不应立刻处理。

### 4. 表单提交与校验
- 监听 `submit` 事件（绑在 form 上，按钮点击或回车触发）后 `e.preventDefault()` 手动处理。
- **原生校验**：required、minlength/maxlength、min/max、pattern（正则）、type="email/url" 等；提交时浏览器自动拦截并显示提示。
- JS 校验 API：`input.validity`（ValidityState：valueMissing、patternMismatch、tooShort……）、`input.checkValidity()` 返回布尔、`input.reportValidity()` 显示浏览器提示、`form.novalidate` 关闭原生校验全权交给 JS。
- 提交方式对比：原生 submit 会整页跳转；现代做法 preventDefault + fetch 异步提交。

## 二十一、脚本加载与页面生命周期

### 1. script 的 defer 与 async

两者的加载与执行时机对比。

HTML 解析到普通 `<script src>` 会**停止解析 → 下载 → 执行 → 再继续解析**（阻塞）。defer/async 都实现异步下载，区别在执行时机：

| 属性 | 下载 | 执行时机 | 顺序保证 | 适用 |
| --- | --- | --- | --- | --- |
| 无 | 阻塞解析 | 下载完立即执行（阻塞） | 按文档顺序 | 需要立即生效的脚本 |
| `defer` | 并行、不阻塞 | HTML 解析完成后、DOMContentLoaded 事件**前** | 按文档顺序执行 | 主脚本（推荐） |
| `async` | 并行、不阻塞 | 下载完立即执行（谁先下完谁先跑） | **无顺序保证** | 独立的统计/广告脚本 |

- 内联 script 不支持 defer/async；`type="module"` 默认具有 defer 行为（可再加 async）。
- 动态创建的 script 默认等效 async，可通过 `script.async = false` 恢复顺序。

### 2. 页面生命周期：DOMContentLoaded、load、beforeunload、unload

```js
document.addEventListener('DOMContentLoaded', fn);  // DOM 树构建完成（不等图片/样式），defer 脚本之后触发
window.addEventListener('load', fn);                // 全部资源（图片、iframe）加载完成
window.addEventListener('beforeunload', fn);        // 离开前，可提示未保存（需 returnValue）
window.addEventListener('unload', fn);              // 正在离开，只能做同步清理（建议用 sendBeacon 发数据）
```

- DOMContentLoaded 会被样式表后的脚本阻塞：脚本会等待其前面的样式表加载完。
- readyState 与 readystatechange：`loading → interactive（≈ DOMContentLoaded 前）→ complete`，可用于「不知道注册时机」的场景。
- 数据上报离开时机推荐 `navigator.sendBeacon(url, data)`（不阻塞卸载）。

### 3. 资源加载：onload、onerror

img、script、link 等资源元素都支持 load / error 事件，用于检测加载成败与兜底：

```js
img.onload = () => console.log(img.width);
img.onerror = () => { img.src = 'fallback.png'; };
```

- script 加载失败的错误无法被 window.onerror 完整捕获跨域细节，需 `crossorigin` 属性 + CORS 才能看到报错详情。
- `window.onerror(message, source, lineno, colno, error)` 与 `window.addEventListener('error', fn)` 捕获运行时错误；资源加载错误也会冒出 error 事件（捕获阶段监听可区分）。

## 二十二、BOM 与浏览器存储

### 1. window、location、history、navigator

- **window**：全局对象兼窗口接口；弹窗 `alert / confirm / prompt`（阻塞式，生产少用）；`window.open(url)` / `close()`；`innerWidth/innerHeight`。
- **location**：`href`（读写跳转）、`search`（查询串）、`hash`、`pathname`、`origin`；`location.assign/replace/reload`；`URLSearchParams` 解析查询串。
- **history**：`back/forward/go(n)`；SPA 核心 `pushState/replaceState`（改 URL 不刷新）+ `popstate` 事件（前进后退触发）。
- **navigator**：`userAgent`、`clipboard`（剪贴板 API）、`language`、`onLine`、`geolocation`、`serviceWorker`。
- 跨窗口通信：`window.open` 返回引用，配合 `postMessage(data, targetOrigin)` + `message` 事件实现 iframe / 新窗口间安全通信。

### 2. cookie 与 document.cookie

- cookie 由服务器 `Set-Cookie` 下发（或 JS 写入 `document.cookie`），之后**每次同源请求自动携带**在请求头中。
- `document.cookie = 'name=tom; max-age=3600; path=/; domain=xx.com; secure; samesite=lax'`——一次只能写一条，读取返回全部 `k=v; k2=v2` 字符串。
- 属性：`Expires/Max-Age`（不设为会话 cookie）、`Path/Domain`（作用范围）、`Secure`（仅 HTTPS）、`HttpOnly`（JS 不可读，防 XSS 窃取）、`SameSite`（Strict/Lax/None，防 CSRF）。
- 局限：容量约 4KB、随请求发送影响性能、API 原始难用——一般业务数据改用 Web Storage。

### 3. localStorage 与 sessionStorage

| 维度 | localStorage | sessionStorage | cookie |
| --- | --- | --- | --- |
| 生命周期 | 永久（手动清除） | 当前标签页会话 | 可设置过期 |
| 容量 | 约 5MB | 约 5MB | 4KB |
| 随请求发送 | 否 | 否 | 是 |
| API | `setItem/getItem/removeItem/clear` | 同左 | 字符串拼接 |

- 值只能是字符串，存对象需 `JSON.stringify/parse`；读写是同步阻塞的，大数据量慎用。
- 同源共享：localStorage 跨标签页共享（storage 事件可监听其他标签页的修改），sessionStorage 仅限本标签页。
- 典型用途：localStorage 存主题/登录态/token，sessionStorage 存表单草稿、一次性状态。

### 4. IndexedDB

浏览器内置的**事务型 NoSQL 数据库**，容量大（可达数百 MB 以上）、支持索引和游标、API 全异步（事件风格或 Promise 包装）。

- 基本流程：`indexedDB.open(name, version)` → `onupgradeneeded` 中建对象仓库（createObjectStore）与索引 → 事务（`db.transaction(store, 'readwrite')`）→ `store.put/get/add/delete`、游标遍历。
- 适用场景：离线应用缓存大量结构化数据（配合 Service Worker 做 PWA）；简单的键值需求用 localStorage 即可。
- 现代封装：idb、Dexie.js 库可大幅简化回调。

→ 展开见 [[web前端开发/WebAPIs/BOM APIs|BOM APIs]]

---

# 第三部分：网络与其他

## 二十三、网络请求

### 1. Fetch

基于 Promise 的现代网络请求 API：

```js
const res = await fetch(url, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(data),
  credentials: 'same-origin',   // 'include' 跨域带 cookie
  signal: controller.signal     // AbortController 取消请求
});
if (!res.ok) throw new Error(`${res.status}`);   // 关键：fetch 只在网络层失败时 reject
const data = await res.json();   // 还有 text() / blob() / formData() / arrayBuffer()
```

- **易错点**：HTTP 404/500 不会让 Promise reject（res.ok 为 false），只有断网等网络错误才 reject；body 只能读一次；默认不带跨域 cookie。
- GET 不能有 body；`Content-Type: application/json` 会触发 CORS 预检（非简单请求）。

### 2. FormData 与文件上传

```js
const fd = new FormData(form);          // 收集整个表单
fd.append('file', fileInput.files[0]);  // 添加文件
fd.append('note', 'extra');
await fetch('/upload', { method: 'POST', body: fd });
// 不要手动设置 Content-Type，浏览器自动生成带 boundary 的 multipart/form-data
```

- FormData 的值可被 spread 成 entries；服务端接收 multipart 编码。
- 上传进度：fetch 尚无官方上传进度，用 XHR 的 `upload.onprogress`，或 fetch 的 `duplex: 'half'` 流式（新）。
- 文件选择：`<input type="file">` 的 `files` 列表、拖拽的 `e.dataTransfer.files`。

### 3. 跨源请求与 CORS

浏览器同源策略：协议、域名、端口任一不同即为跨源，默认禁止跨源读取响应（img/script 标签加载不受限）。

- **简单请求**（GET/HEAD/POST 且头部受控）直接发送，服务器需返回 `Access-Control-Allow-Origin` 浏览器才把响应交给 JS。
- **预检请求（preflight）**：非简单请求（如带自定义头、PUT/DELETE、application/json）先发 `OPTIONS` 询问，服务器回应 `Access-Control-Allow-Origin / -Methods / -Headers / -Max-Age` 后才发真正请求。
- 携带 cookie：前端 `credentials: 'include'`，服务器 `Access-Control-Allow-Credentials: true` 且 Allow-Origin 不能为 `*`。
- 无 CORS 的替代：JSONP（script 标签，仅 GET，已过时）、开发环境代理（vite proxy / nginx 反向代理，浏览器视角同源）。

### 4. XMLHttpRequest

fetch 之前的主流方案，事件风格，仍用于进度监听等场景：

```js
const xhr = new XMLHttpRequest();
xhr.open('GET', url);
xhr.timeout = 5000;
xhr.responseType = 'json';
xhr.onload = () => console.log(xhr.status, xhr.response);   // 也可 onreadystatechange + readyState
xhr.onerror = () => {};
xhr.upload.onprogress = (e) => console.log(e.loaded / e.total);  // 上传进度
xhr.send();
xhr.abort();   // 取消
```

- `readyState`：0 未发送 → 1 已 open → 2 收到响应头 → 3 下载中 → 4 完成；`status` HTTP 状态码。
- 相比 fetch：进度事件成熟、可同步（不推荐）、历史兼容；代码冗长，新项目首选 fetch/axios。

### 5. WebSocket 与 Server-Sent Events

**WebSocket**：全双工、长连接，独立协议（ws:// wss://），服务器可主动推送。

```js
const ws = new WebSocket('wss://example.com/chat');
ws.onopen = () => ws.send(JSON.stringify({ type: 'join' }));
ws.onmessage = (e) => console.log(e.data);    // 文本或 Blob/ArrayBuffer
ws.onclose = (e) => console.log(e.code, e.reason);
ws.close();
// 心跳保活 + 指数退避重连是生产标配
```

**SSE（Server-Sent Events）**：基于 HTTP 的**单向**服务器推送，自动重连、文本流：

```js
const es = new EventSource('/events');
es.onmessage = (e) => console.log(e.data);
es.addEventListener('custom', fn);   // 服务端 event: custom
```

- 选型：需要双向（聊天、协作、游戏）用 WebSocket；只需要服务器推送（行情、通知、打字机效果）SSE 更简单，兼容 HTTP 基础设施。

### 6. URL 对象

```js
const url = new URL('https://a.com:8080/p?q=1#h');
url.protocol; url.host; url.pathname; url.hash;
url.searchParams.get('q');                 // URLSearchParams：get/has/append/set/delete
url.searchParams.set('page', '2');
url.toString();
```

- `searchParams` 免去手写 `encodeURIComponent` 的解析拼串；fetch、`new Image().src` 等都可直接接受 URL 对象（自动 toString）。
- 编码函数区分：`encodeURIComponent`（用于查询参数值）与 `encodeURI`（整体 URL，保留保留字符）。

## 二十四、二进制数据与文件

### 1. ArrayBuffer 与类型化数组

**ArrayBuffer** 是固定长度的原始二进制内存块，不能直接读写，需通过「视图」操作：

- **类型化数组（TypedArray）**：Int8Array / Uint8Array / Int16Array / Float32Array / Float64Array 等 9 种（加上 BigInt64Array / BigUint64Array 共 11 种），每个元素固定位宽，语法与普通数组相似（没有 push/splice）。
- **DataView**：按偏移自由读写任意类型，可指定**字节序**（`true` 为小端），处理协议解析更灵活。

```js
const buf = new ArrayBuffer(16);
const view = new Uint8Array(buf);
view[0] = 255;
const dv = new DataView(buf);
dv.getInt32(0, true);
```

- 应用：WebGL、音视频处理、canvas 像素、文件解析、WebSocket 二进制帧。
- 相互转换：`new TextEncoder().encode(str)`（字符串→UTF-8 Uint8Array）、`TextDecoder` 反向；Blob/arrayBuffer/file 之间用 `await blob.arrayBuffer()`。

### 2. Blob、File 与 FileReader

- **Blob**（Binary Large Object）：不可变的二进制数据对象，可含 MIME 类型；`new Blob([data], {type: 'text/html'})`；`blob.slice()` 分片、`blob.text()/arrayBuffer()/stream()` 读取。
- **File** 继承 Blob，加了 name/lastModified，来自 `<input type="file">` 和拖拽。
- **FileReader**：异步读取 File/Blob：

```js
const reader = new FileReader();
reader.onload = (e) => console.log(e.target.result);
reader.readAsText(file);            // 文本
reader.readAsDataURL(file);         // base64，用于 <img src> 预览
reader.readAsArrayBuffer(file);     // 二进制处理
reader.onprogress / onerror
```

- **ObjectURL**：`URL.createObjectURL(blob)` 生成 `blob:` 短链（更快、不需 base64 编码），用完 `URL.revokeObjectURL` 释放；下载文件惯用法：a 标签 + objectURL + download 属性。

## 二十五、动画与 Web Components

### 1. JavaScript 动画与 requestAnimationFrame

- `requestAnimationFrame(callback)`：在**下一次重绘前**执行回调，浏览器自动对齐刷新率（60fps 时约 16.7ms 一次），页面隐藏时自动暂停，是 JS 动画的正确方式；返回 id，`cancelAnimationFrame(id)` 取消。

```js
function animate(duration, step) {
  const start = performance.now();
  requestAnimationFrame(function frame(now) {
    const progress = Math.min((now - start) / duration, 1);
    step(progress);            // 缓动函数可在此应用
    if (progress < 1) requestAnimationFrame(frame);
  });
}
```

- 与 setInterval 对比：rAF 与渲染同步不丢帧、不耗电；setTimeout 动画频率不稳且后台仍执行。
- 优先级：能用 CSS transition/animation（合成器线程驱动，性能最好）就不用 JS；JS 动画适合与交互状态强耦合的场景。
- rAF 回调属于渲染前执行，不属于宏任务/微任务，顺序上在一次宏任务之后的渲染管线中。

### 2. Web Components：Custom Elements 与 Shadow DOM

浏览器原生的组件化三件套，无需框架：

```js
class MyCard extends HTMLElement {
  connectedCallback() {          // 插入文档时
    const shadow = this.attachShadow({ mode: 'open' });   // Shadow DOM：样式与结构隔离
    shadow.innerHTML = `
      <style> p { color: steelblue; } </style>
      <p><slot></slot></p>`;     // slot 插槽接收 light DOM 内容
  }
}
customElements.define('my-card', MyCard);
// 使用：<my-card>内容</my-card>
```

- **Custom Elements**：`customElements.define(tag, class)` 注册自定义标签；生命周期 connectedCallback / disconnectedCallback / attributeChangedCallback / adoptedCallback。
- **Shadow DOM**：`attachShadow` 创建隔离的渲染树——内部样式不外泄、外部样式不影响（结构隔离），`slot` 做内容分发。
- **template 与 slot**：`<template>` 内容不渲染、可克隆复用，`<slot>` 定义内容出口。
- 与 React/Vue 对比：原生标准、可跨框架复用、样式真隔离；但生态、数据绑定、SSR 支持弱于框架，常用于设计系统/微前端的自定义元素输出。

---

上级：[[web前端开发/web前端开发|web前端开发]]

## 笔记

- [[web前端开发/JavaScript/高频使用的方法|高频使用的方法]]
- [[web前端开发/JavaScript/面试题目|面试题目]]
- [[web前端开发/面试准备/八股/JavaScript八股文（标准版）|JavaScript 八股文（标准版）]]
- [[web前端开发/面试准备/八股/JavaScripts八股文（手写版）|JavaScript 八股文（手写版）]]
- [[web前端开发/WebAPIs/WebAPIs|WebAPIs]]
