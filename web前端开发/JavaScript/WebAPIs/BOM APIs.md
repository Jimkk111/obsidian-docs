---
title: BOM APIs
source: https://www.yuque.com/mook-mvqcp/wc9wsu/ay5kgzrb9686g442
created: 2026-04-28
updated: 2026-04-28
tags:
  - JavaScript
---

### 一、window全局对象

浏览器环境，window全局对现有两个作用：一是作为全局API的挂载点、二是全局Global对象。

**（一）物理屏幕、浏览器窗口、布局视口、可视视口相关属性、方法**

（1）物理屏幕

window.screen对象是物理屏幕的抽象描述。

window.devicePixelRatio，设备像素比，指物理像素和逻辑像素的比例（通常情况下，单位物理像素要比单位逻辑像素要小），与每英寸像素数DPI对应。

（2）浏览器窗口

浏览器窗口指浏览器应用程序窗口大小，包含标签栏、菜单栏、可视窗口等内容。

window.outerWidth、window.outerHeight，整个浏览器窗口的外尺寸（包含地址栏、标签栏、滚动条等）

window.screenTop/screenY、window.screenLeft/screenX，浏览器窗口左上角相对于屏幕的坐标

window.resizeTo()/resizeBy()，编程式改变浏览器窗口的大小

window.moveTo()/window.moveBy()，编程式改变浏览器窗口的位置

以上两种改变浏览器窗口大小和位置的方法受浏览器安全策略限制。

resize事件，当浏览器窗口大小变化时触发（此时可视视口通常也会改变）

（3）布局视口

布局视口是网页的“画布”，CSS 百分比、vw/vh 等相对单位基于它计算。移动端通常通过 `<meta name="viewport">` 控制宽度。

API

说明

`document.documentElement.clientWidth`

/ `.clientHeight`

布局视口的宽高（不含滚动条，是最可靠的布局视口尺寸）

`document.documentElement.scrollWidth`

/ `.scrollHeight`

整个可滚动内容的尺寸（可能大于布局视口）

`document.body.clientWidth`

/ `.clientHeight`

在怪异模式下可能使用；标准模式下建议用 `document.documentElement`

`window.scrollX`

/ `window.pageXOffset`

文档在水平/垂直方向的滚动距离（相对于布局视口）

`window.scrollY`

/ `window.pageYOffset`

同上

`window.scrollTo()`

, `scrollBy()`

, `scroll()`

滚动布局视口（改变可视视口显示的区域）

（4）可视视口

可视视口是用户实际可见的区域，受缩放、键盘弹出、滚动条影响。

window.visualViewport是可视视口对象，包含以下属性：

resize事件，当可视视口尺寸变化（软键盘弹出/收起、缩放）时触发

scroll事件，可视视口滚动时触发（通常与 `window.scroll` 同步，但更精细）

window.innerWidth/window.innerHeight，历史兼容方式，默认返回可视视口尺寸（当滚动条存在时可能包含滚动条宽度，且不区分缩放，建议用 `visualViewport` 替代）

**（二）打开新窗口**

（1）window.open ()

window.open ()方法可以用于导航到指定URL,也可以用于打开新浏览器窗口。这个方法接收4个参数:要加载的URL、目标窗口、特性字符串和表示新窗口在浏览器历史记录中是否替代当前加载页面的布尔值。

这个方法返回一个对新窗口的引用，这个对象和普通window对象没什么区别。

与该方法配套的是close()方法，这个方法在新窗口的对象上，用于关闭新窗口自己，关闭后引用还在，但只能访问closed属性。

新创建窗口的window对象有一个opener属性,指向打开它的窗口。这个属性只在弹出窗口的最上层windoW对象(top)有定义,是指向调用window.open ()打开它的窗口或窗格的指针

**（三）location、navigator、history**

**1.location对象**

window和document都有指向location对象的引用。

（1）location包含当前窗口加载的文档的信息，主要是与url有关的信息。

（2）location.assign()

接收一个url字符串，导航到新url，并新加一条历史记录。

（3）location.replace()

接收一个url字符串，导航到新url，但是不会新增历史记录，而是替代 当前的历史记录，同时不能回到前一页。

（4）location.reload()

重新加载页面。如果传入了布尔值true，表示重新向服务器加载；如果不传参数，可能从缓存读取。

**2.navigator对象**

**3.history对象**

history对象用于管理同一标签页下的历史记录（每一个标签页都有一个独立的history对象）。值得一提的是，它是实现spa路由的基础。

（1）属性

history.length表示历史记录栈的历史条目总数

history.state，当前历史条目关联的状态对象（通过 `pushState` / `replaceState` 设置）。若未设置则为 `null`。

（2）导航方法go()、back()、forward()

这些用于跳转历史条目，加载对应的历史页面.

`history.back()`

后退一页（等同于用户点击浏览器后退按钮）

`history.forward()`

前进一页

`history.go(delta)`

相对当前页移动 `delta`

步。
正数向前，负数向后。
`go(0)`

或 `go()`

刷新当前页

`history.go(-1)`

等同于 `back()`

`history.go(1)`

等同于 `forward()`

（3）pushstate()和replacestate()

pushstate()用于向历史记录栈添加新的历史条目但不刷新页面。

replacestate()替换指定历史条目，同样不会刷新页面。

（4）popstate事件

当活动历史条目发生变化时触发（用户点击前进/后退按钮，或调用 `back()`/`forward()`/`go()`）。注意：`pushState`/`replaceState` 不会触发此事件。事件回调可通过 `event.state` 获取关联的状态对象。

**（四）文档对象document**

**（五）客户端存储**

**1.cookie**

cookie用于在客户端存储会话信息，

cookie的set-Cookie HTTP头部包含会话信息，格式为name=value（set-Cookie还包含cookie的参数）

cookie存储在客户端的浏览器，不仅仅存储着name、value，同时存储各种参数，包含过期时间、cookie类型、域、路径、安全标志等等。

cookie有许多限制：

**JavaScript操作cookie**

document.cookie属性是处理cookie的唯一方法。

当读取该属性时，返回页面中所以有效cookie的字符串，以分号分隔。由于cookie使用url编码（**url编码也称百分号编码，为了传输非ASCII编码的字符，需要将其使用url编码方式编码。**），所以需要使用decodeURLComponent()解码。

当给这个属性赋值时，可以添加一个新cookie而不会覆盖已有的所有cookie，除非设置了已有的cookie（覆盖同名的cookie）。新cookie最好使用encodeURLComponent()编码。

**2.sessionsStorage**

Storage类型用于保存名/值对，直至达到存储上限，sessionsStorage和localStorage都是Storage类型，都可以使用以下属性或方法：

Storage对象发生改变时，会触发storage事件，事件对象包含domain、key、newValue、oldValue四个属性。

sessionsStorage的特点：

**3.localStorage**

localStorage与sessionsStorage的区别是，localStorage存储的名/值对持久的，即关闭浏览器后这些数据仍然被保存，直至通过JavaScript删除或者浏览器删除缓存。注意要访问同一个localStorage对象，页面必须来自同一个源。

**4.indexedDB**

浏览器非关系型数据库，以对象（关系型数据库是表即table）为存储结构存储数据。

概念：实例、请求、数据库、对象存储、记录、事务、游标、游标范围、索引。

**（六）JSON**

JSON（JavaScript Object Notation），是在互联网传输数据的一种标准（XML也是一种数据传输标准，但是使用起来较为麻烦），表现为一种数据结构。

JSON是一个通用数据格式，不仅可以在JavaScript使用，很多语言都有解析和序列化Json字符串的内置能力。

**1.JSON语法**

JSON支持简单类型、对象和数组三种类型的值。

JSON的字符串必须使用双引号包裹。

属性必须使用双引号包裹。

**2.序列化**

JSON.stringify()方法完成将JavaScript数据序列化为JSON字符串。

这个方法有三个参数，第一个参数是数据，第二个参数是过滤器，第三个参数是指定缩进。

对象可以包含一个toJSON()方法，指定序列化的方式。

toJsON()方法可以与过滤函数一起用,因此理解不同序列化流程的顺序非常重要。在把对象传给JSON.stringify()时会执行如下步骤：

(1)如果可以获取实际的值,则调用toJsON()方法获取实际的值,否则使用默认的序列化。

(2)如果提供了第二个参数,则应用过滤。传入过滤函数的值就是第(1)步返回的值。

(3)第(2)步返回的每个值都会相应地进行序列化。(4)如果提供了第个参数,则相应地进行缩进。

**3.解析**

JSON.parse方法实现将JSON字符串解析为JavaScript数据。

与序列化相似，解析过程也可以设置解析选项。
