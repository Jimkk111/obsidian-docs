---
title: Vue教程
source: https://www.yuque.com/mook-mvqcp/wc9wsu/yoctd5gxlfg480nu
created: 2026-05-24
updated: 2026-05-24
tags:
  - Vue
---

# Vue2

### 创建vue实例

vue2创建vue实例的方式是new Vue({...})，一般这样创建的是应用实例

组件实例和vue实例大致一样，只不过vue实例是根实例。当定义一个组件（`.vue` 文件或全局组件），本质上是在描述一个 可复用的 Vue 实例配置。当根组件渲染时，遇到 `<Child />` 标签，Vue 就会创建该子组件的独立实例，形成父子关系。

**对比vue3**：Vue3中创建vue实例的方式是createApp({...}).mounted("#app")。

# Vue3响应式系统原理

响应式系统的核心功能是实现起数据更新时触发副作用函数自动执行，对于有渲染功能的副作用函数来说，就是数据更新时触发视图重新渲染。

响应式系统的实现涉及副作用函数注册为依赖、响应式数据注册、管理依赖集合的数据结构...

## （一）基本的响应式系统实现

### 1.注册副作用函数为依赖

注册副作用函数为依赖的函数是effect(fn)，fn是真正的副作用函数即内部包含对响应式数据的操作。effect内部的主要操作是：首先将activeEffect变量的值设为fn，然后执行fn（触发代理拦截set从而被追踪为依赖）。

### 2.响应式数据注册

Vue3的响应式数据注册基于proxy实现，proxy提供了对对象操作的拦截，从而能够重写set和get等对象操作的逻辑。

当读取对象的值时，在set里面根据activeEffect追踪副作用函数为依赖，即把activeEffect加入到bucket中；当设置对象的值时，把key对应的所有副作用函数从bucket中取出重新执行。

追踪副作用函数、重写执行副作用函数的代码被分别封装为了track(target,key)和trigger(target,key)两个函数。

### 3.管理依赖集合的数据结构

Vue3用bucket来管理依赖集合，这是一个树形的结构。bucket是一个WeakMap，键是原始对象target，值是一个Map实例，这个Map实例的键是原始对象target的key，值是一个由副作用函数组成的Set。
