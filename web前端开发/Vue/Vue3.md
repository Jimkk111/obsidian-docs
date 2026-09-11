---
title: Vue3
created: 2026-09-07
updated: 2026-09-07
tags:
  - Vue
---

> [!info] 目录说明
> 目录结构参照 Vue3 官方文档（[cn.vuejs.org/guide](https://cn.vuejs.org/guide/)）的章节体系，并补齐面试高频主题。标注「待补充」的小节为尚未填充内容，按需逐步完善。

## 一、基础

### 1. 创建应用实例（待补充）

`createApp({...}).mount("#app")`、应用实例与组件实例。

### 2. 模板语法与指令（待补充）

插值、指令、动态参数、修饰符。

### 3. 响应式核心：ref 与 reactive（待补充）

ref 与 reactive 的区别、toRef / toRefs、shallowRef / shallowReactive、响应式转换的注意事项。

### 4. 计算属性 computed（待补充）

### 5. 侦听器 watch 与 watchEffect（待补充）

watch 的侦听来源、deep / immediate、watchEffect 与 watch 的区别。

### 6. Class 与 Style 绑定（待补充）

### 7. 条件渲染与列表渲染（待补充）

v-if 与 v-show、v-for 与 key。

### 8. 事件处理与表单绑定（待补充）

事件修饰符、v-model 在组件上的用法。

## 二、组件

### 1. 组件通信（待补充）

props / emit、v-model 双向绑定组件、provide / inject 依赖注入。

### 2. 插槽 slot（待补充）

默认插槽、具名插槽、作用域插槽。

### 3. 生命周期（待补充）

setup 中的生命周期钩子（onMounted 等）、父子组件执行顺序。

### 4. 组合式函数 Composables（待补充）

自定义组合式函数的封装规范与最佳实践。

### 5. 内置组件（待补充）

Teleport、Suspense、KeepAlive、Transition。

### 6. setup 语法糖（待补充）

`<script setup>` 的特性、defineProps / defineEmits / defineExpose。

## 三、响应式系统原理

响应式系统的核心功能是实现数据更新时触发副作用函数自动执行，对于有渲染功能的副作用函数来说，就是数据更新时触发视图重新渲染。

响应式系统的实现涉及副作用函数注册为依赖、响应式数据注册、管理依赖集合的数据结构等。

### 1. 注册副作用函数为依赖

注册副作用函数为依赖的函数是 effect(fn)，fn 是真正的副作用函数即内部包含对响应式数据的操作。effect 内部的主要操作是：首先将 activeEffect 变量的值设为 fn，然后执行 fn（触发代理拦截 set 从而被追踪为依赖）。

### 2. 响应式数据注册

Vue3 的响应式数据注册基于 Proxy 实现，Proxy 提供了对对象操作的拦截，从而能够重写 set 和 get 等对象操作的逻辑。

当读取对象的值时，在 get 里面根据 activeEffect 追踪副作用函数为依赖，即把 activeEffect 加入到 bucket 中；当设置对象的值时，把 key 对应的所有副作用函数从 bucket 中取出重新执行。

追踪副作用函数、重新执行副作用函数的代码被分别封装为了 track(target, key) 和 trigger(target, key) 两个函数。

（待补充：effect 的调度与执行时机、ref 的实现、响应式系统的边界情况）

### 3. 管理依赖集合的数据结构

Vue3 用 bucket 来管理依赖集合，这是一个树形的结构。bucket 是一个 WeakMap，键是原始对象 target，值是一个 Map 实例，这个 Map 实例的键是原始对象 target 的 key，值是一个由副作用函数组成的 Set。

## 四、路由与状态管理

### 1. Vue Router（待补充）

→ 详见 [[web前端开发/Vue/Vue Router|Vue Router]]

### 2. Pinia（待补充）

→ 详见 [[web前端开发/Vue/Pinia|Pinia]]

## 五、常见面试问题

### 1. Vue2 与 Vue3 的区别（待补充）

响应式原理（defineProperty vs Proxy）、API 风格（Options vs Composition）、性能优化（编译时优化、静态提升、patch flag）、生命周期变化、Fragment 支持。

### 2. ref 和 reactive 的区别（待补充）

### 3. Composition API 的优势（待补充）

### 4. 为什么 Vue3 用 Proxy 替代 Object.defineProperty（待补充）

### 5. v-if 和 v-for 的优先级（待补充）

---

上级：[[web前端开发/Vue/Vue教程|Vue教程]]

## 笔记

- [[web前端开发/Vue/关于Vue.js细枝末节的原理|关于Vue.js细枝末节的原理]]
