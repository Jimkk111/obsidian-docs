---
title: CSS
source: https://www.yuque.com/mook-mvqcp/wc9wsu/lng0rgnbdbfm366v
created: 2026-05-25
updated: 2026-05-25
tags:
  - HTML+CSS
---

**1.grid布局**

（1）基础概念：

- 网格线：划分行列的分界线

- 网格轨道，两条网格线围成的区域

- 网格单元：一条列轨道和一条行轨道交叉形成的区域

- 网格区域，任意四条网格线围成的封闭区域

- 网格容器，开启display:grid的元素

- 网格项，网格容器的直接子元素

（2）样式和函数、关键字

display:grid - 开启网格布局

grid-template-rows、grid-template-columns -定义网格行和网格列

grid-auto-rows、grid-auto-columns，隐式行、隐式列

repeat()，用于声明多个尺寸相同的行或列

auto-fill和auto-fit，当不确定一行或者一列的数量时，用做repeat()的第一个参数。

minmax()，声明一个数值范围，第一个参数是最小值，第二个参数是最大值

fr - 剩余空间的比例单位

grid-row、grid-column，用在网格项目，声明跨行、跨列

grid-auto-flow - 决定隐式网格的 排列方向 以及是否启用 密集填充（`dense`）

gap、gap-row、gap-column

（3）对齐

justify表示水平方向，align表示垂直方向。place是简写，先垂直后水平。

整个网格在容器内的对齐：justify-content、align-content，写在容器的选择器。

单个项目在单元格内的对齐：justify-items、align-items，写在容器的选择器，控制所有单元格；justify-self、align-self，写在项目的选择器，只作用这一个单元格。

（4）应用

grid布局实现不了瀑布流，瀑布流用多列布局实现。

**2.多列布局**

column-count - 声明列数

column-width - 声明列宽

column- gap - 列间隙

column - rule 列分隔线，类似border属性

**3.图片**

**4.背景**

filter属性是滤镜属性，其取值blur()表示模糊半径，可以搭配inset使用（作用是把元素拉大到大于背景，因为blur()需要参考附件像素计算模糊，如果大小和背景一样大，靠边界的区域就没有模糊效果。），打造盖在背景上的模糊层

**5.定义**

inset属性很实用，是top、bottom、right和left，可以控制定位元素的大小。
