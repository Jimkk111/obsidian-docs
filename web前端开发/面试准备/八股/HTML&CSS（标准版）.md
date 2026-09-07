---
title: HTML&CSS（标准版）
source: https://www.yuque.com/mook-mvqcp/wc9wsu/iftvlnsk86fak8tg
created: 2026-05-28
updated: 2026-05-28
tags:
  - 面试准备
---

### 1.说说你对盒模型的理解

盒模型是浏览器对页面元素的结构的形象化表示。每个元素在页面上就好像表现为一个矩形的盒子，从里到外分别是内容区、内边距、边框和外边距。内容区存放实际的内容，宽度和高度默认控制的是这块区域；内边距是边框和内容区之间的透明区域；边框是包裹在内边距外面的实线或虚线框，可以设置粗细、样式和颜色；外边距位于盒子最外层，用于控制当前盒子与相邻盒子之间的间距，也是透明的。

盒模型分为标准盒模型和怪异盒模型，通过设置盒子的box-sizing属性自由切换两种模型。box-sizing:content-box为标准盒模型，这是默认值；box-sizing:border-box是怪异盒模型。二者的主要区别是计算width和height的方式不同。标准盒模型的width和height只包含内容区；怪异盒模型的width和height包含内容区+内边距+边框。理解这两种盒子模型有利于更精准地控制元素的位置和尺寸，避免莫名溢出的问题。

### 2.css选择器有哪些？优先级？哪些属性可以继承？

CSS选择器可以分为以下5大类：

（1）基础选择器：通配符选择器、标签选择器、类选择器、ID选择器。

（2）关系选择器：后代选择器、子选择器、相邻兄弟选择器、通用兄弟选择器。

（3）伪类选择器：动态伪类比如:hover、:active、:focus，结构伪类比如:first-child、:last-child、:nth-child(n)、:nth-of-type()，否定伪类:not()，还有表示根元素的:root，表示无子元素的元素:empty。

（4）伪元素选择器：作用是创建虚拟元素，包含::before、::after、::first-line、::first-letter等等。

（5）属性选择器：用一对中括号包裹着一个属性名，或者同时指定属性名对应的属性值，比如[type="text"]，选择输入框。

优先级按从高到底排序为!import、内联样式、ID选择器、类选择器和伪类选择器和属性选择器、元素选择器和伪元素选择器、通配符选择器和关系选择器和否定伪类自身、继承的属性。

元素可以继承的属性主要为字体类（比如font-size、font-family、font-style等等）、文本类（比如color、text-indent、letter-spacing、text-decoration等）、列表类（比如list-style、list-style-position等等）的属性，不可继承的属性主要与盒模型、定位以及背景相关。

### 3.有哪些方式可以隐藏页面元素，有什么区别？
