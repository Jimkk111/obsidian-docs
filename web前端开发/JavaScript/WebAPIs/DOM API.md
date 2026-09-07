---
title: DOM API
source: https://www.yuque.com/mook-mvqcp/wc9wsu/gwug0u71sgvn3gcq
created: 2026-04-26
updated: 2026-04-26
tags:
  - JavaScript
---

**document和元素结点其实包含的方法有很多相似的地方，但是前者因为是全局对象，所以包含一些特殊的API。**

**1.获取元素的所有属性**

element.getAttribute()

**2.获取元素的子元素**

element.children，返回HTML集合类型，只读属性。

**3.查询元素**

document.querySelector()或者是document.querySelectorAll()。不只是document才有这两个方法，普通的结点也有，含义是以这个结点为根向下查询。

**4.删除、创建、添加、替换、移动DOM结点的方式**

（1）创建新元素

document.createElement(tagName)，创建元素结点

document.createTextNode(text)，创建文本结点

document.createDocumentFragment()，创建文档片段，批量添加优化

cloneNode(deep)，克隆已有结点（true代表深克隆）

（2）插入到DOM树

parent.appendChild(child)，作为最后一个结点添加

parent.prepend(child)，作为第一个结点添加

parent.append(...nodesOrStrings)，在尾部一次性添加多个结点

`child.before(...nodes)`/ `childafter(...)`，在当前节点前/后插入兄弟节点（现代）

parent.replaceChild(newChild, oldChild)，替代现有结点

（3）修改内容

方式

适用场景

`element.textContent = '新文本'`

纯文本内容（推荐，安全）

`element.innerText = '新文本'`

类似但会触发重排/考虑样式

`element.innerHTML = '<b>带标签</b>'`

解析 HTML 片段（有 XSS 风险）

`element.outerHTML = '<div>...</div>'`

替换整个元素及其内容

`document.title = '新标题'`

直接修改标签页标题

`input.value = '新值'`

表单控件值

（4）删除、替换结点

方法

说明

`element.remove()`

删除自身（现代）

`parent.removeChild(child)`

父节点删除子节点（传统）

`parent.replaceChild(newChild, oldChild)`

替换

`child.replaceWith(...nodes)`

用其他节点替换自身（现代）

清空所有子节点：`while(parent.firstChild) parent.removeChild(parent.firstChild);`

高效清空

`parent.innerHTML = ''`

快速清空（但可能内存泄漏）

（4）移动结点

```javascript

// 将现有节点移动到新位置（自动从原位置移除）
parent.appendChild(existingNode);
parent.insertBefore(existingNode, refNode);
parent.prepend(existingNode);

```

**5.控制元素的样式、类名**

```javascript

// 直接修改 style 对象
element.style.backgroundColor = 'red';
element.style.cssText = 'color: blue; font-size: 14px';

// 操作类名
element.classList.add('active');
element.classList.remove('hidden');
element.classList.toggle('open');
element.classList.replace('old', 'new');

// 获取/设置计算样式
getComputedStyle(element).color;

// className 整体替换（旧方式）
element.className = 'class1 class2';

```
