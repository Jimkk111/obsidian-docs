---
title: Vue2
created: 2026-09-07
updated: 2026-09-07
tags:
  - Vue
---

> [!info] 目录说明
> 目录结构参照 Vue2 官方文档（[v2.cn.vuejs.org/v2/guide](https://v2.cn.vuejs.org/v2/guide/)）的章节体系，并补齐面试高频主题。标注「待补充」的小节为尚未填充内容，按需逐步完善。

## 一、基础

### 1. 创建 Vue 实例

Vue2 创建 Vue 实例的方式是 `new Vue({...})`，一般这样创建的是应用实例。

组件实例和 Vue 实例大致一样，只不过 Vue 实例是根实例。当定义一个组件（`.vue` 文件或全局组件），本质上是在描述一个可复用的 Vue 实例配置。当根组件渲染时，遇到 `<Child />` 标签，Vue 就会创建该子组件的独立实例，形成父子关系。

**对比 Vue3**：Vue3 中创建 Vue 实例的方式是 `createApp({...}).mount("#app")`。

### 2. 模板语法（待补充）

插值（文本、原始 HTML、特性、JS 表达式）、常用指令（v-text、v-html、v-bind、v-on）。

### 3. 计算属性与侦听器（待补充）

computed 的缓存特性、watch 的使用、computed 与 watch 的区别。

### 4. Class 与 Style 绑定（待补充）

对象语法、数组语法。

### 5. 条件渲染与列表渲染（待补充）

v-if / v-else-if / v-else 与 v-show 的区别、v-for 与 key 的作用、数组与对象的响应式注意事项（Vue.set）。

### 6. 事件处理（待补充）

内联处理器与方法、事件修饰符（.stop、.prevent、.once 等）、按键修饰符。

### 7. 表单输入绑定 v-model（待补充）

v-model 的本质（value + input 事件的语法糖）、修饰符（.lazy、.number、.trim）。

### 8. 组件基础（待补充）

props 传参与校验、自定义事件、$emit、组件上使用 v-model。

## 二、深入组件

### 1. 组件通信方式（待补充）

props / $emit、$refs、事件总线 EventBus、provide / inject、$attrs / $listeners、Vuex。

### 2. 插槽 slot（待补充）

默认插槽、具名插槽、作用域插槽。

### 3. 动态组件与 keep-alive（待补充）

component :is、keep-alive 的 include / exclude、activated / deactivated 钩子。

### 4. 生命周期（待补充）

beforeCreate → created → beforeMount → mounted → beforeUpdate → updated → beforeDestroy → destroyed；父子组件生命周期的执行顺序。

### 5. mixin 混入（待补充）

混入规则、与组件选项的合并策略、mixin 的问题。

### 6. 自定义指令与渲染函数（待补充）

## 三、过渡与动画（待补充）

transition / transition-group 的使用、常见动画场景。

## 四、Vue2 响应式原理（待补充）

Object.defineProperty 的劫持方式与缺陷（无法监听新增/删除属性、数组下标）、依赖收集（Dep 与 Watcher）、$set / $delete 的实现、数组方法重写。

## 五、路由与状态管理

### 1. Vue Router（待补充）

→ 详见 [[web前端开发/Vue/Vue Router|Vue Router]]

### 2. Vuex（待补充）

state、getter、mutation、action、module。

## 六、常见面试问题

### 1. v-if 和 v-show 的区别（待补充）

### 2. computed 和 watch 的区别（待补充）

### 3. key 的作用与原理（待补充）

### 4. Vue2 双向绑定原理（待补充）

→ 详见 [[#四、Vue2 响应式原理]]

### 5. Vue2 与 Vue3 的区别（待补充）

---

上级：[[web前端开发/Vue/Vue教程|Vue教程]]
