---
title: Vue2教程
source: https://www.yuque.com/mook-mvqcp/wc9wsu/yoctd5gxlfg480nu
created: 2026-05-24
updated: 2026-09-08
tags:
  - Vue
  - Vue2
---

# Vue2 教程

> 参考官方文档：[Vue2 官方文档（v2.cn.vuejs.org）](https://v2.cn.vuejs.org/v2/guide/)
> Vue3 相关内容见 [[web前端开发/Vue/Vue3教程|Vue3教程]]

上级：[[web前端开发/Vue/Vue|Vue]]

## 目录

1. [[#介绍与安装]]
2. [[#Vue 实例]]
3. [[#模板语法]]
4. [[#计算属性和侦听器]]
5. [[#Class 与 Style 绑定]]
6. [[#条件渲染]]
7. [[#列表渲染]]
8. [[#事件处理]]
9. [[#表单输入绑定]]
10. [[#组件基础]]
11. [[#深入了解组件]]
12. [[#过渡与动画]]
13. [[#可复用性与组合（混入/自定义指令/插件/过滤器）]]
14. [[#Vue2 响应式原理]]
15. [[#生命周期]]
16. [[#生态（Vue Router / Vuex）]]

---

## 介绍与安装

Vue.js 是一套用于构建用户界面的**渐进式框架**，核心库只关注视图层，易于与第三方库或既有项目整合。

声明式渲染：只需声明好模板与数据的关系，Vue 会自动把数据渲染到 DOM 上。

```html
<div id="app">{{ message }}</div>
```

```js
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

安装方式：

```bash
# CDN
<script src="https://cdn.jsdelivr.net/npm/vue@2"></script>

# NPM
npm install vue@2

# CLI（Vue CLI，Vue2 时代的官方脚手架）
npm install -g @vue/cli
vue create my-project
```

---

## Vue 实例

### 创建vue实例

vue2创建vue实例的方式是new Vue({...})，一般这样创建的是应用实例

组件实例和vue实例大致一样，只不过vue实例是根实例。当定义一个组件（`.vue` 文件或全局组件），本质上是在描述一个 可复用的 Vue 实例配置。当根组件渲染时，遇到 `<Child />` 标签，Vue 就会创建该子组件的独立实例，形成父子关系。

**对比vue3**：Vue3中创建vue实例的方式是createApp({...}).mount("#app")。

### 实例常用选项

```js
new Vue({
  el: '#app',        // 挂载目标
  data,              // 数据对象（Vue2 中必须是一个对象，组件中必须是函数）
  methods: {},       // 方法
  computed: {},      // 计算属性
  watch: {},         // 侦听器
  props: {},         // 组件接收的外部数据
  components: {},    // 局部注册的组件
  created() {},      // 生命周期钩子
})
```

### data 必须是函数（组件中）

组件的 `data` 必须是一个函数，这样每个实例可以维护一份被返回对象的独立的拷贝：

```js
data: function () {
  return { count: 0 }
}
```

---

## 模板语法

Vue2 使用基于 HTML 的模板语法，将 DOM 绑定至底层 Vue 实例的数据。

### 插值

```html
<!-- 文本插值 -->
<span>{{ message }}</span>

<!-- v-once：只执行一次性插值，不再响应变化 -->
<span v-once>{{ message }}</span>

<!-- 原始 HTML（XSS 风险：只对可信内容使用） -->
<div v-html="rawHtml"></div>

<!-- 特性绑定 -->
<div v-bind:id="dynamicId"></div>
```

### 指令

指令带有 `v-` 前缀，其职责是当表达式的值改变时，将其产生的连带影响响应式地作用于 DOM。

```html
<!-- 参数：v-bind / v-on -->
<a v-bind:href="url">链接</a>
<a v-on:click="doSomething">点击</a>

<!-- 缩写 -->
<a :href="url">链接</a>
<a @click="doSomething">点击</a>

<!-- 动态参数（2.6.0+） -->
<a v-bind:[attributeName]="url">链接</a>
```

修饰符（modifier）如 `.prevent`、`.stop`、`.sync`、`.lazy` 等：

```html
<form v-on:submit.prevent="onSubmit"></form>
```

---

## 计算属性和侦听器

### computed

计算属性基于它们的响应式依赖进行**缓存**，只在相关依赖发生改变时才会重新求值。

```js
computed: {
  fullName: {
    // getter
    get() { return this.firstName + ' ' + this.lastName },
    // setter（可选）
    set(newValue) {
      const names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
```

对比 methods：methods 每次调用都会重新执行；computed 基于依赖缓存。

### watch

侦听器用于在数据变化时执行异步或开销较大的操作：

```js
watch: {
  question(newVal, oldVal) {
    this.getAnswer()
  }
}
```

> 原则：能用 computed 的场景优先用 computed；watch 更适合异步/开销大的操作。

---

## Class 与 Style 绑定

```html
<!-- 对象语法 -->
<div v-bind:class="{ active: isActive, 'text-danger': hasError }"></div>

<!-- 数组语法 -->
<div v-bind:class="[activeClass, errorClass]"></div>

<!-- 绑定内联样式 -->
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

---

## 条件渲染

```html
<div v-if="type === 'A'">A</div>
<div v-else-if="type === 'B'">B</div>
<div v-else>C</div>

<!-- v-show：总是渲染，只是切换 display；不适合包含 v-for -->
<h1 v-show="ok">Hello!</h1>
```

- `v-if` 是**真正**的条件渲染，切换时事件监听器和子组件被销毁重建；有更高切换开销。
- `v-show` 基于初始渲染，切换开销小；适合频繁切换。
- `v-if` 与 `v-for` 一起使用时，`v-for` 具有比 `v-if` 更高的优先级（不推荐同时使用）。

---

## 列表渲染

```html
<ul id="example">
  <li v-for="(item, index) in items" :key="item.id">
    {{ index }} - {{ item.message }}
  </li>
</ul>

<!-- 遍历对象 -->
<div v-for="(value, key, index) in object">{{ key }}: {{ value }}</div>

<!-- 遍历数字 -->
<span v-for="n in 10">{{ n }}</span>
```

数组更新检测：Vue2 对数组使用**变异方法**（push、pop、shift、unshift、splice、sort、reverse）能检测到变化，但通过索引直接 `vm.items[index] = newValue` 或修改 length **无法检测**，需使用：

```js
Vue.set(vm.items, index, newValue)   // 或 vm.$set
vm.items.splice(index, 1, newValue)
```

`key` 的作用：给每个节点一个唯一标识，便于 diff 算法高效更新，尽量不用 index 作为 key。

---

## 事件处理

```html
<!-- 直接执行方法 / 内联语句 -->
<button v-on:click="greet">Greet</button>
<button v-on:click="say('hi')">Say hi</button>

<!-- 事件修饰符 -->
<a v-on:click.stop="doThis"></a>       <!-- 阻止冒泡 -->
<form v-on:submit.prevent="onSubmit"></form>  <!-- 阻止默认行为 -->
<!-- 其他：.capture .self .once .passive -->

<!-- 按键修饰符 -->
<input v-on:keyup.enter="submit">
<input v-on:keyup.13="submit">

<!-- 系统修饰键 -->
<input v-on:click.ctrl="doSomething">

<!-- 表单输入修饰符 -->
<input v-model.lazy="msg">   <!-- change 事件同步 -->
<input v-model.number="age"> <!-- 转为数字 -->
<input v-model.trim="msg">   <!-- 去除首尾空白 -->
```

---

## 表单输入绑定

`v-model` 本质是语法糖，负责监听用户的输入事件以更新数据，并对极端场景做一些特殊处理。

```html
<input v-model="message" placeholder="编辑我">
<textarea v-model="message"></textarea>

<!-- 复选框：单个绑定布尔值，多个绑定数组 -->
<input type="checkbox" v-model="checked">
<input type="checkbox" value="Jack" v-model="checkedNames">

<!-- 单选按钮 / 选择框 -->
<input type="radio" value="A" v-model="picked">
<select v-model="selected"><option>A</option></select>

<!-- 值绑定到动态特性 -->
<input type="checkbox" :true-value="a" :false-value="b" v-model="toggle">
<input type="radio" :value="someValue" v-model="pick">
```

---

## 组件基础

### 组件注册

```js
// 全局注册
Vue.component('my-component-name', { /* ... */ })

// 局部注册
new Vue({
  components: {
    'component-a': ComponentA
  }
})
```

### 通过 Prop 向子组件传递数据

```js
Vue.component('blog-post', {
  props: ['title']
  // 或带校验：
  // props: { title: { type: String, required: true, default: '' } }
})
```

```html
<blog-post title="My journey"></blog-post>
<!-- 动态 prop -->
<blog-post :title="post.title"></blog-post>
```

Prop 是单向数据流：不应该在子组件中直接修改 prop。

### 通过事件向父组件发送消息

```js
// 子组件
this.$emit('my-event', value)
```

```html
<blog-post @my-event="onMyEvent"></blog-post>
```

`.sync` 修饰符（双向绑定 prop 的语法糖）：

```html
<text-document :title.sync="doc.title"></text-document>
<!-- 等价于 -->
<text-document :title="doc.title" @update:title="doc.title = $event"></text-document>
```

### 组件上使用 v-model

自定义组件上的 `v-model` 利用 `value` prop 和 `input` 事件实现：

```js
Vue.component('custom-input', {
  props: ['value'],
  template: `<input :value="value" @input="$emit('input', $event.target.value)">`
})
```

### 通过插槽分发内容

```html
<todo-item>
  待办事项文本（默认插槽内容）
</todo-item>
```

```js
Vue.component('todo-item', {
  template: `<li><slot></slot></li>`
})
```

---

## 深入了解组件

### 插槽详解

```html
<!-- 具名插槽 -->
<base-layout>
  <template v-slot:header><h1>header</h1></template>
  <p>默认插槽</p>
  <template #footer><p>footer</p></template>
</base-layout>

<!-- 作用域插槽：子组件向插槽内容传数据 -->
<slot-list :items="items">
  <template #default="{ item }">{{ item.text }}</template>
</slot-list>
```

```js
// 子组件
this.$slots.default        // 访问默认插槽
this.$scopedSlots.item     // 作用域插槽（2.6.0 后统一）
```

### 动态组件与异步组件

```html
<!-- 动态组件：currentTabComponent 可以是已注册组件名或组件选项对象 -->
<component v-bind:is="currentTabComponent"></component>

<!-- keep-alive 缓存组件状态 -->
<keep-alive><component v-bind:is="currentTabComponent"></component></keep-alive>
```

```js
// 异步组件
Vue.component('async-example', function (resolve, reject) {
  setTimeout(() => resolve(require('./my-async-component')), 500)
})
// 工厂函数返回 Promise（2.3.0+ / webpack 动态 import）
Vue.component(
  'async-webpack-example',
  () => import('./my-async-component')
)
```

### 处理边界情况

- 访问根实例 / 父组件实例：`this.$root`、`this.$parent`
- 访问子组件实例或子元素：`ref` 与 `this.$refs`
- 依赖注入：`provide` / `inject`（祖先向所有后代注入数据）
- 程序化的事件监听器：`this.$on`、`this.$once`、`this.$off`
- 递归组件：组件在自身模板中调用自身（需有 `name`）
- 循环引用、控制更新（`forceUpdate`、`v-once`）

---

## 过渡与动画

Vue 提供了 `transition` 和 `transition-group` 封装组件，在插入、更新或移除 DOM 时应用过渡效果。

```html
<transition name="fade">
  <p v-if="show">hello</p>
</transition>
```

```css
.fade-enter-active, .fade-leave-active { transition: opacity .5s; }
.fade-enter, .fade-leave-to /* 2.1.8+ */ { opacity: 0; }
```

过渡类名：`v-enter`（起始）、`v-enter-active`（过程）、`v-enter-to`（结束）、`v-leave`、`v-leave-active`、`v-leave-to`。

`<transition-group>` 用于列表过渡，内部元素需提供 `key`。

---

## 可复用性与组合（混入/自定义指令/插件/过滤器）

### 混入（mixin）

```js
var myMixin = {
  created: function () { this.hello() },
  methods: { hello: function () { console.log('hello from mixin!') } }
}
new Vue({ mixins: [myMixin] })
```

- 数据对象会**递归合并**，组件数据优先（同名冲突时以组件为准）。
- 同名钩子函数将合并为一个数组，**混入对象的钩子先执行**。
- 值为对象的选项（methods、components、directives）合并为同一对象，冲突时组件优先。

### 自定义指令

```js
// 注册全局指令
Vue.directive('focus', {
  inserted: function (el) { el.focus() }
})

// 钩子函数：bind / inserted / update / componentUpdated / unbind
```

### 插件

```js
MyPlugin.install = function (Vue, options) {
  Vue.myGlobalMethod = function () {}
  Vue.directive('my-directive', {})
  Vue.prototype.$myMethod = function () {}
}
Vue.use(MyPlugin)
```

### 过滤器

```html
{{ message | capitalize }}
<div v-bind:id="rawId | formatId"></div>
```

```js
// 全局
Vue.filter('capitalize', function (value) {
  if (!value) return ''
  return value.charAt(0).toUpperCase() + value.slice(1)
})
// 局部：filters 选项
```

> Vue3 中过滤器已被移除，可用计算属性或方法代替。

### 渲染函数与 JSX

```js
Vue.component('anchored-heading', {
  render: function (createElement) {
    return createElement(
      'h' + this.level,
      this.$slots.default
    )
  },
  props: { level: { type: Number, required: true } }
})
```

---

## Vue2 响应式原理

Vue2 的响应式基于 `Object.defineProperty` 实现（区别于 Vue3 的 Proxy，见 [[web前端开发/Vue/Vue3教程#深入响应式系统|Vue3教程]]）。

```js
Object.defineProperty(obj, key, {
  get() { /* 依赖收集：watcher 收集到 Dep */ return val },
  set(newVal) { val = newVal; /* 派发更新：通知 Dep 中所有 watcher */ }
})
```

局限性：

1. 无法检测对象属性的**添加和删除**（需要 `Vue.set` / `Vue.delete`）。
2. 无法检测**数组索引赋值**和修改 length（需要重写数组变异方法）。
3. 需要递归遍历初始化所有属性，初始化开销大。

核心结构：每个响应式属性对应一个 `Dep`（依赖收集器），每个组件实例对应一个渲染 `Watcher`；getter 中收集依赖，setter 中派发更新（`dep.notify()`），配合异步批处理（`nextTick` 与调度队列）完成视图更新。

---

## 生命周期

```
beforeCreate -> created -> beforeMount -> mounted
                                        -> beforeUpdate -> updated
                                        -> beforeDestroy -> destroyed
```

| 钩子 | 时机 | 常见用途 |
| --- | --- | --- |
| created | 实例创建完成，数据观测完成，但未挂载 | 初始化数据、发请求 |
| mounted | 实例挂载到 DOM | 操作 DOM、初始化第三方库 |
| updated | 数据更新导致视图重渲染后 | 依赖 DOM 的后续操作 |
| destroyed | 实例销毁后 | 清除定时器、解绑自定义事件 |

其他：`activated` / `deactivated`（keep-alive）、`errorCaptured`（2.5.0+）。

---

## 生态（Vue Router / Vuex）

- [[web前端开发/Vue/Vue Router|Vue Router]]：官方路由，Vue2 对应 Vue Router 3.x。
- [[web前端开发/Vue/Pinia|Pinia]] / Vuex 3.x：状态管理，Vue2 时代主流是 Vuex。

## 相关笔记

- [[web前端开发/Vue/Vue3教程|Vue3教程]]
- [[web前端开发/Vue/关于Vue.js细枝末节的原理|关于Vue.js细枝末节的原理]]
