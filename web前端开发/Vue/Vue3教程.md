---
title: Vue3教程
source: https://www.yuque.com/mook-mvqcp/wc9wsu/yoctd5gxlfg480nu
created: 2026-09-08
updated: 2026-09-08
tags:
  - Vue
  - Vue3
---

# Vue3 教程

> 参考官方文档：[Vue3 官方文档（cn.vuejs.org）](https://cn.vuejs.org/guide/introduction)
> Vue2 相关内容见 [[web前端开发/Vue/Vue2教程|Vue2教程]]

上级：[[web前端开发/Vue/Vue|Vue]]

## 目录

1. [[#介绍与快速上手]]
2. [[#创建 Vue 应用]]
3. [[#组合式 API 基础]]
4. [[#响应式基础]]
5. [[#计算属性与侦听器]]
6. [[#模板语法]]
7. [[#Class 与 Style 绑定]]
8. [[#条件渲染]]
9. [[#列表渲染]]
10. [[#事件处理]]
11. [[#表单输入绑定]]
12. [[#生命周期]]
13. [[#组件（Props / 事件 / 插槽 / 依赖注入）]]
14. [[#内置组件]]
15. [[#可复用性（组合式函数 / 自定义指令 / 插件）]]
16. [[#深入响应式系统]]
17. [[#生态（Vue Router / Pinia）]]

---

## 介绍与快速上手

Vue 是一款用于构建用户界面的 JavaScript 框架，基于标准 HTML、CSS 和 JavaScript 构建。

Vue3 的核心特性：

- **组合式 API**（Composition API）：更好的逻辑组织与复用。
- **性能提升**：Proxy 响应式、编译时优化（静态提升、patch flag、Block Tree）。
- **更好的 TypeScript 支持**。
- 新内置组件：`Teleport`、`Suspense`、`Fragment`（多根节点模板）。

创建应用：

```bash
npm create vue@latest   # 官方脚手架 create-vue
# 或使用 Vite 手动创建
npm create vite@latest my-vue-app -- --template vue
```

---

## 创建 Vue 应用

每个 Vue 应用都是通过 `createApp` 函数创建一个新的**应用实例**开始的：

```js
import { createApp } from 'vue'

const app = createApp({
  /* 根组件选项 */
})

// 挂载应用（生产版本应使用 innerHTML 而非 innerText）
app.mount('#app')
```

对比 Vue2（见 [[web前端开发/Vue/Vue2教程#Vue 实例|Vue2教程]]）：Vue2 通过 `new Vue({...})` 创建根实例，Vue3 改为 `createApp` 创建应用实例，再 `.mount('#app')` 挂载。应用实例上还可以注册全局资源：

```js
app.component('TodoDeleteButton', TodoDeleteButton) // 全局组件
app.directive('focus', FocusDirective)             // 全局指令
app.use(MyPlugin)                                  // 插件
```

---

## 组合式 API 基础

组合式 API 在单文件组件中通常与 `<script setup>` 搭配使用：

```vue
<script setup>
import { ref, onMounted } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}

onMounted(() => console.log(`初始计数为 ${count.value}。`))
</script>

<template>
  <button @click="increment">计数：{{ count }}</button>
</template>
```

- `<script setup>` 中的顶层绑定（变量、函数、import）都会自动暴露给模板。
- 相比 Vue2 的选项式 API（Options API），组合式 API 允许将同一逻辑关注点的代码组织在一起，并方便抽取为可复用的**组合式函数**。

---

## 响应式基础

### ref

`ref()` 接收任意类型的值，返回一个带 `.value` 属性的响应式引用：

```js
import { ref } from 'vue'

const count = ref(0)
console.log(count.value) // 0
count.value++
```

- 在 JS 中访问 / 修改需要 `.value`；在模板中会**自动解包**（顶层属性）。
- `shallowRef()`：只对 `.value` 的替换本身做响应式，不做深层代理。

### reactive

`reactive()` 返回对象的响应式代理（基于 Proxy）：

```js
import { reactive } from 'vue'

const state = reactive({ count: 0 })
state.count++
```

局限：

1. 只对**对象类型**有效（对象、数组、Map、Set），对原始类型无效。
2. 替换整个对象会导致响应式连接丢失。
3. 解构属性或将其传给函数时会丢失响应性（解构出的原始值不再被追踪）。

### 选择

- 默认使用 `ref()`，逻辑更清晰、不依赖解构限制。
- `reactive()` 适合集中管理一组相关状态的场景。

---

## 计算属性与侦听器

### computed

```js
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  get: () => firstName.value + ' ' + lastName.value,
  set: (newValue) => {
    [firstName.value, lastName.value] = newValue.split(' ')
  }
})
```

计算属性会基于依赖**缓存**，只有依赖变化时才重新计算；与 Vue2 一致。

### watch 与 watchEffect

```js
import { ref, watch, watchEffect } from 'vue'

const x = ref(0)

// watch：显式指定数据源，惰性执行，能拿到旧值
watch(x, (newVal, oldVal) => {
  console.log(`x 由 ${oldVal} 变为 ${newVal}`)
})

// watch 多个来源
watch([x, () => x.value * 2], ([x1, x2]) => console.log(x1, x2))

// watchEffect：立即执行并自动追踪依赖，无法获取旧值
const unwatch = watchEffect(() => console.log(`x is ${x.value}`))
unwatch() // 停止侦听
```

watch 回调默认在组件更新**之前**执行；`flush: 'post'` 选项可改为 DOM 更新后执行。

---

## 模板语法

Vue3 模板语法与 Vue2 类似：文本插值 `{{ }}`、原始 HTML `v-html`、特性绑定 `v-bind`、事件 `v-on`、动态参数 `v-bind:[name]`。

Vue3 中模板支持**多根节点**（Fragment），不再要求单根元素。

指令的完整语法：`v-指令名:参数.修饰符="表达式"`。

---

## Class 与 Style 绑定

用法与 Vue2 相同，支持对象语法、数组语法：

```vue
<template>
  <div :class="{ active: isActive }"></div>
  <div :class="[activeClass, errorClass]"></div>
  <div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
</template>
```

---

## 条件渲染

```vue
<template>
  <div v-if="type === 'A'">A</div>
  <div v-else-if="type === 'B'">B</div>
  <div v-else>C</div>

  <h1 v-show="ok">Hello!</h1>
</template>
```

注意：Vue3 中 `v-if` 的优先级**高于** `v-for`（与 Vue2 相反），因此 `v-if` 中无法访问 `v-for` 的变量，不推荐同时使用。

---

## 列表渲染

```vue
<template>
  <li v-for="(item, index) in items" :key="item.id">
    {{ index }} - {{ item.message }}
  </li>
</template>
```

- 基于 Proxy 的响应式系统可以**直接检测**数组索引赋值、length 修改、对象属性新增 / 删除，不再需要 `Vue.set` / `Vue.delete`（Vue3 中已移除这两个 API）。
- `:key` 推荐使用唯一 id，避免使用 index。

---

## 事件处理

```vue
<template>
  <button @click="greet">Greet</button>
  <button @click="count++">+1</button>

  <!-- 事件修饰符 -->
  <a @click.stop.prevent="doThat"></a>
  <!-- 其他：.capture .self .once .passive -->

  <!-- 按键修饰符 -->
  <input @keyup.enter="submit" />
  <input @keyup.page-down="onPageDown" />

  <!-- 系统修饰键 / .exact -->
  <button @click.ctrl.exact="onCtrlClick">A</button>
</template>
```

Vue3 中移除了 `@keyup.13` 数字键码写法和 `$on` / `$off` 实例事件总线 API（跨组件通信推荐使用 mitt 等外部库或 provide/inject）。

---

## 表单输入绑定

`v-model` 用法与 Vue2 相同。组件上的 `v-model` 在 Vue3 中有变化：

- 默认绑定 `modelValue` prop，接收 `update:modelValue` 事件：
  `<MyComponent v-model="title" />` 等价于 `<MyComponent :modelValue="title" @update:modelValue="newTitle => title = newTitle" />`
- 支持**多个 v-model** 绑定与自定义参数名：`<MyComponent v-model:title="bookTitle" />`
- 支持自定义修饰符（如 `v-model.capitalize`），通过 `modelModifiers` prop 获取。

---

## 生命周期

组合式 API 中的生命周期钩子以 `on` 开头，需要在 `setup()` 中同步注册：

```js
import { onMounted, onUpdated, onUnmounted } from 'vue'

onMounted(() => { /* DOM 挂载后 */ })
onUpdated(() => { /* 响应式数据变化后 DOM 更新完毕 */ })
onUnmounted(() => { /* 组件卸载后，清理副作用 */ })
```

完整流程（组合式 API 写法）：

```
setup（相当于 beforeCreate/created 合并）
-> onBeforeMount -> onMounted
-> onBeforeUpdate -> onUpdated
-> onBeforeUnmount -> onUnmounted
```

与 Vue2 差异：`beforeCreate` / `created` 由 `setup()` 本身替代；`beforeDestroy` / `destroyed` 改名为 `onBeforeUnmount` / `onUnmounted`。

---

## 组件（Props / 事件 / 插槽 / 依赖注入）

### Props

```vue
<script setup>
defineProps({
  title: String,
  likes: { type: Number, default: 0 },
  isPublished: { type: Boolean, required: true }
})
</script>
```

Prop 仍是单向数据流：子组件不应直接修改 prop。

### 自定义事件

```vue
<script setup>
const emit = defineEmits(['enlarge-text'])
function submit() {
  emit('enlarge-text', 0.1)
}
</script>
```

```vue
<MyComponent @enlarge-text="onEnlargeText" />
```

组件事件的校验：`defineEmits` 也可以传对象，值为校验函数。

### 插槽

```vue
<!-- 子组件 -->
<div class="container">
  <header><slot name="header"></slot></header>
  <main><slot></slot></main>
  <footer><slot name="footer"></slot></footer>
</div>

<!-- 父组件 -->
<BaseLayout>
  <template #header><h1>标题</h1></template>
  <template #default><p>默认内容</p></template>
  <template #footer><p>页脚</p></template>
</BaseLayout>
```

作用域插槽：

```vue
<!-- 子组件 -->
<slot name="item" :item="item" :index="index"></slot>

<!-- 父组件 -->
<MyList>
  <template #item="{ item, index }">{{ index }}. {{ item.text }}</template>
</MyList>
```

### 透传属性（Fallthrough Attributes）

未被子组件声明为 props / emits 的属性（如 class、id、事件）会自动透传到子组件根元素；多根节点组件没有自动透传，需通过 `useAttrs()` 显式绑定。

### 依赖注入（provide / inject）

解决深层嵌套组件传 props 的"逐级透传"问题：

```vue
<!-- 祖先组件 -->
<script setup>
import { provide, ref } from 'vue'
const location = ref('North Pole')
provide('location', location)          // 提供响应式数据
</script>

<!-- 后代组件 -->
<script setup>
import { inject } from 'vue'
const location = inject('location', '默认值')
</script>
```

大型应用中推荐以 Symbol 作为注入 key，并保持提供方负责修改（将修改方法一并 provide）。

---

## 内置组件

| 组件 | 作用 |
| --- | --- |
| `<Transition>` | 单元素 / 组件的进入、离开过渡动画 |
| `<TransitionGroup>` | 列表过渡动画 |
| `<KeepAlive>` | 缓存组件实例，切换时保留状态，配合 `include` / `exclude` / `max` |
| `<Teleport>` | 将组件渲染到 DOM 树的其他位置（如弹窗挂到 body） |
| `<Suspense>` | 协调对异步组件的依赖，处理加载中 / 失败状态（实验性） |

```vue
<Teleport to="body">
  <div class="modal">弹窗内容</div>
</Teleport>

<KeepAlive :include="['MyComponent']">
  <component :is="currentComponent" />
</KeepAlive>
```

---

## 可复用性（组合式函数 / 自定义指令 / 插件）

### 组合式函数（Composables）

约定以 `use` 开头，封装并复用**有状态的逻辑**（Vue2 时代用 mixin 解决，但 mixin 存在命名冲突、来源不清晰等问题）：

```js
// useMouse.js
import { ref, onMounted, onUnmounted } from 'vue'

export function useMouse() {
  const x = ref(0)
  const y = ref(0)
  function update(e) {
    x.value = e.pageX
    y.value = e.pageY
  }
  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))
  return { x, y }
}
```

```vue
<script setup>
import { useMouse } from './useMouse'
const { x, y } = useMouse()
</script>
```

### 自定义指令

```js
const focus = {
  mounted: (el) => el.focus()
}

// 局部：<script setup> 中以 vFocus 使用
const vFocus = focus

// 全局注册
app.directive('focus', focus)
```

钩子函数：`created` / `beforeMount` / `mounted` / `beforeUpdate` / `updated` / `beforeUnmount` / `unmounted`。

### 插件

```js
export default {
  install(app, options) {
    app.config.globalProperties.$translate = (key) => /* ... */
    app.provide('i18n', options)
    app.directive('my-directive', {})
  }
}
```

```js
app.use(MyPlugin, { greeting: 'hello' })
```

---

## 深入响应式系统

响应式系统的核心功能是实现起数据更新时触发副作用函数自动执行，对于有渲染功能的副作用函数来说，就是数据更新时触发视图重新渲染。

响应式系统的实现涉及副作用函数注册为依赖、响应式数据注册、管理依赖集合的数据结构...

### 1.注册副作用函数为依赖

注册副作用函数为依赖的函数是effect(fn)，fn是真正的副作用函数即内部包含对响应式数据的操作。effect内部的主要操作是：首先将activeEffect变量的值设为fn，然后执行fn（触发代理拦截set从而被追踪为依赖）。

### 2.响应式数据注册

Vue3的响应式数据注册基于proxy实现，proxy提供了对对象操作的拦截，从而能够重写set和get等对象操作的逻辑。

当读取对象的值时，在get里面根据activeEffect追踪副作用函数为依赖，即把activeEffect加入到bucket中；当设置对象的值时，把key对应的所有副作用函数从bucket中取出重新执行。

追踪副作用函数、重写执行副作用函数的代码被分别封装为了track(target,key)和trigger(target,key)两个函数。

### 3.管理依赖集合的数据结构

Vue3用bucket来管理依赖集合，这是一个树形的结构。bucket是一个WeakMap，键是原始对象target，值是一个Map实例，这个Map实例的键是原始对象target的key，值是一个由副作用函数组成的Set。

### 补充：整体流程

```
effect(fn) 保存 activeEffect = fn 并执行 fn
  -> fn 中读取响应式数据 -> Proxy get 拦截 -> track(target, key)
     -> bucket: WeakMap<target, Map<key, Set<effect>>> 中收集 activeEffect
  -> 修改响应式数据 -> Proxy set 拦截 -> trigger(target, key)
     -> 取出该 key 对应的所有副作用函数并重新执行
```

与 Vue2 的对比：

| | Vue2 | Vue3 |
| --- | --- | --- |
| 实现方式 | `Object.defineProperty` 劫持 getter/setter | `Proxy` 代理整个对象 |
| 对象属性增删 | 无法检测，需 `Vue.set` / `Vue.delete` | 原生支持（Proxy 的 deleteProperty 等） |
| 数组索引赋值 | 无法检测，需重写变异方法 | 原生支持 |
| 初始化开销 | 需递归遍历所有属性 | 惰性代理，访问时才追踪 |
| 数据结构 | Dep + Watcher | WeakMap -> Map -> Set 的 bucket |

更多细节见 [[web前端开发/Vue/关于Vue.js细枝末节的原理|关于Vue.js细枝末节的原理]]。

### 常用响应式 API

```js
import { reactive, ref, readonly, shallowReactive, shallowRef, toRef, toRefs, unref, isRef, isReactive, isProxy } from 'vue'
```

---

## 生态（Vue Router / Pinia）

- [[web前端开发/Vue/Vue Router|Vue Router]]：Vue3 对应 Vue Router 4.x。
- [[web前端开发/Vue/Pinia|Pinia]]：Vue3 官方推荐的状态管理库（Vuex 5 已由 Pinia 取代）。

## 相关笔记

- [[web前端开发/Vue/Vue2教程|Vue2教程]]
- [[web前端开发/Vue/关于Vue.js细枝末节的原理|关于Vue.js细枝末节的原理]]
