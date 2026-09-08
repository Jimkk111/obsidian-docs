---
title: Pinia
source: https://www.yuque.com/mook-mvqcp/wc9wsu/adl1ipdepy6rsg6i
created: 2026-04-18
updated: 2026-09-08
tags:
  - Vue
---

# Pinia

> 参考官方文档：[Pinia 官方文档](https://pinia.vuejs.org/zh/introduction.html)
> Pinia 同时支持 Vue2 与 Vue3，API 基本一致。本文以 Vue3 写法为主线，Vue2 的差异集中标注（见 [[#五、Vue2 与 Vue3 环境差异]]）。

上级：[[web前端开发/Vue/Vue|Vue]]

## 目录

1. [[#一、简介]]
2. [[#二、使用流程]]
3. [[#三、核心概念]]
4. [[#四、对比Vuex]]
5. [[#五、Vue2 与 Vue3 环境差异]]
6. [[#六、其他（SSR / 测试）]]

---

### 一、简介

Pinia是Vue官方的状态管理库，实现跨组件或者页面共享状态。

**什么时候应该使用Store(Pinia)?**

Store应该存储在整个应用中访问的数据。

### 二、使用流程

以Vue3为例。

1.安装Pinia

```bash
npm install pinia
```

2.导入createPinia创建Pinia实例，并将其传递给应用

```js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const pinia = createPinia()  // 这个pinia实例是store实例的容器，是根。
const app = createApp(App)

app.use(pinia)
app.mount('#app')
```

> Vue2 的注册方式见 [[#五、Vue2 与 Vue3 环境差异]]。

3.定义store

无论是Option Store还是Setup Store，都是使用defineStore来定义一个store。

defineStore()方法返回一个创建store实例的方法（用useStore来指代，因为这个名字由用户而定），手动调用useStore才能创建store实例。

在执行defineStore时Option会被转换为Setup形式。

(1)Option Store

```js
export const useCounterStore = defineStore('counter', {
  // option store的state用rective处理，所以下面的方法不需要用.value调用state
  state: () => ({ count: 0, name: 'Eduardo' }),
  getters: {
    doubleCount: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++
    },
  },
})
```

(2)Setup Store

```js
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0) // state用ref处理，所以下面的方法要通过.value调用state
  const name = ref('Eduardo')
  const doubleCount = computed(() => count.value * 2)
  function increment() {
    count.value++
  }

  return { count, name, doubleCount, increment }
})
```

> Setup Store 中应返回应用中需要的所有状态，只包含 ref / computed / function；Vue2 中 `ref` / `computed` 需从 `@vue/composition-api` 导入。

4.使用store

手动调用useCounterStore获取store实例。

```vue
<script setup>
  import { useCounterStore } from '@/stores/counter'
  // 在组件内部的任何地方均可以访问变量 `store` ✨
  const store = useCounterStore()
</script>
```

Vue2 选项式组件中：

```vue
<script>
import { useCounterStore } from '@/stores/counter'

export default {
  computed: {
    count() {
      return useCounterStore().count
    },
  },
  methods: {
    increment() {
      useCounterStore().increment()
    },
  },
}
</script>
```

**从store解构要注意的细节**

useStore方法创建一个被reactive()包裹的store实例，解构store实例会丢失state的响应性， storeToRefs(store)可以解决这个问题，它将为每一个响应式属性创建引用。

```vue
<script setup>
import { storeToRefs } from 'pinia'
const store = useCounterStore()
// state / getter 解构为响应式的 ref
const { count, doubleCount } = storeToRefs(store)
// action 可以直接解构
const { increment } = store
</script>
```

### 三、核心概念

**1.state**

**(1)访问state**

store.*访问，并且允许正常修改

**(2)重置state**

store.$reset()，将state设置为初始值。

在Setup Store中需要自己定义$reset

**(3)选项式API中访问state、可修改的state**

mapState(storeName，stateName)方法的作用是在选项式API里方便地映射state为只读的计算属性，不用每次都要调用store.*访问state，简化操作。stateName可以是一个数组，也可以是一个对象（重命名和自定义逻辑）。

如果想修改这些 state 属性 (例如，如果有一个表单)，可以使用 `mapWritableState()` 作为代替。但注意不能像 `mapState()` 那样传递一个函数。

map 辅助函数完整示例（Vue2 选项式组件中同样常用）：

```js
import { mapState, mapWritableState, mapActions } from 'pinia'
import { useCounterStore } from '@/stores/counter'

export default {
  computed: {
    ...mapState(useCounterStore, ['count', 'doubleCount']),
    ...mapWritableState(useCounterStore, { myCount: 'count' }),
  },
  methods: {
    ...mapActions(useCounterStore, ['increment']),
  },
}
```

**(4)变更state**

store.$patch()，传入一个state的补丁对象或者包含操作state的函数来更改store。

**(5)替换state**

你不能完全替换掉 store 的 state，因为那样会破坏其响应性。但是，你可以 patch 它。

```js
// 这实际上并没有替换`$state`
store.$state = { count: 24 }
// 在它内部调用 `$patch()`：
store.$patch({ count: 24 })
```

你也可以通过变更 `pinia` 实例的 `state` 来设置整个应用的初始 state。这常用于 SSR 中的激活过程。

```js
pinia.state.value = {}
```

**(6)订阅state**

通过 store 的 `$subscribe()` 方法侦听 state 及其变化。比起普通的 `watch()`，使用 `$subscribe()` 的好处是 subscriptions 在 patch 后只触发一次。

**2.getter**

state的计算值。

getter 完全等价于 store 的 state 的计算值（computed），推荐用箭头函数，接收 `state` 作为参数；需要访问 `this` 时用普通函数并注意类型。

```js
export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  getters: {
    // 箭头函数
    doubleCount: (state) => state.count * 2,
    // 依赖其他 getter：通过 this 访问，需标注返回类型
    doublePlusOne(): number {
      return this.doubleCount + 1
    },
    // 向 getter 传参：返回一个函数
    getCountById: (state) => (id) => state.items.find((item) => item.id === id),
  },
})
```

**3.action**

state的method。

action 可以是同步的也可以是异步的（异步即返回 Promise），不再像 Vuex 那样需要 mutation 中转：

```js
export const useUsersStore = defineStore('users', {
  state: () => ({ users: [], loading: false }),
  actions: {
    async fetchUsers() {
      this.loading = true
      try {
        const res = await api.getUsers()
        this.users = res.data
      } finally {
        this.loading = false
      }
    },
  },
})
```

- action 之间可以互相调用（`this.otherAction()`），也可以使用其他 store。
- `$onAction()` 可订阅 action 的调用（可用于埋点、日志、错误上报）。

**4.插件**

Pinia 支持扩展（插件），本质是一个接收 context 的函数，可用于持久化、同步到 localStorage 等：

```js
export function myPiniaPlugin(context) {
  // context: { pinia, app, store, options }
  context.store.subscribe((mutation) => { /* 每次变更触发 */ })
  context.store.$onAction(() => {})
}
// 注册：pinia.use(myPiniaPlugin)
```

### 四、对比Vuex

| | Vuex 4 | Pinia |
| --- | --- | --- |
| mutation | 必须通过 mutation 同步修改 state | **没有 mutation**，action 中直接修改（同步/异步均可） |
| 模块（modules） | 嵌套模块，需要 namespaced 处理命名冲突 | **扁平结构**，一个 store 即一个模块，store 之间可相互引用 |
| TypeScript | 支持薄弱，类型推导困难 | 完整的类型推导 |
| Composition API | 支持有限 | 天然支持（Setup Store 即组合式写法） |
| 重置状态 | 无内置 | `store.$reset()` |
| devtools / 热更新 | 支持 | 支持且更好（时间旅行、模块热更新不丢状态） |
| 体积 | 较大 | 约 1KB，API 更简洁 |

Vue3 官方已推荐 Pinia 取代 Vuex 作为默认状态管理方案；Vue2 时代主流是 Vuex 3.x。

### 五、Vue2 与 Vue3 环境差异

Pinia 的核心 API（defineStore、$patch、$reset、$subscribe、storeToRefs）在两个版本下完全一致，差异集中在安装与响应式底层：

**版本要求**

- Vue **2.6.14+**（不支持 2.5 及以下）。
- 需要 **@vue/composition-api** 插件提供组合式 API。

**安装与注册**

```js
import Vue from 'vue'
import VueCompositionAPI from '@vue/composition-api'
import { createPinia, PiniaVuePlugin } from 'pinia'

// 1. 注册组合式 API（必须在注册 Pinia 之前）
Vue.use(VueCompositionAPI)
// 2. 注册 Pinia 的 Vue2 插件（Vue3 用 app.use(pinia)，无需这一步）
Vue.use(PiniaVuePlugin)

const pinia = createPinia()

new Vue({
  el: '#app',
  pinia, // 将 pinia 传给根实例
  render: (h) => h(App),
})
```

**注意事项**

1. 必须先注册 composition-api 插件，再注册 Pinia，否则运行时报错。
2. Setup Store 的 `ref` / `computed` 从 `@vue/composition-api` 导入，而不是 `vue`。
3. Vue2 响应式系统的限制依然存在：对象**新增属性**、**数组索引赋值**不会触发更新，需 `Vue.set` / `splice`，复杂数据变更建议用 `$patch`。
4. 存量 Vue2 + Vuex 项目不建议为迁移而迁移；Pinia 对 Vue2 的支持更适合"新项目不想写 mutation"的场景。

### 六、其他（SSR / 测试）

- SSR：在服务端每次请求创建独立的 pinia 实例，通过 `pinia.state.value` 激活初始 state；配合 Nuxt 使用更简单。
- 测试：通过 `createTestingPinia()` 创建测试专用实例，可 mock action。

## 相关笔记

- [[web前端开发/Vue/Vue2教程|Vue2教程]]
- [[web前端开发/Vue/Vue3教程|Vue3教程]]
- [[web前端开发/Vue/Vue Router|Vue Router]]
