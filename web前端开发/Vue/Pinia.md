---
title: Pinia
source: https://www.yuque.com/mook-mvqcp/wc9wsu/adl1ipdepy6rsg6i
created: 2026-04-18
updated: 2026-04-18
tags:
  - Vue
---

### 一、简介

Pinia是Vue官方的状态管理库，实现跨组件或者页面共享状态。

**什么时候应该使用Store(Pinia)?**

Store应该存储在整个应用中访问的数据。

### 二、使用流程

以Vue3为例。

1.安装Pinia

2.导入createPinia创建Pinia实例，并将其传递给应用

```vue

import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const pinia = createPinia()  // 这个pinia实例是store实例的容器，是根。
const app = createApp(App)

app.use(pinia)
app.mount('#app')

```

3.定义store

无论是Option Store还是Setup Store，都是使用defineStore来定义一个store。

defineStore()方法返回一个创建store实例的方法（用useStore来指代，因为这个名字由用户而定），手动调用useStore才能创建store实例。

在执行defineStore时Option会被转换为Setup形式。

(1)Option Store

```javascript

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

```javascript

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

4.使用store

手动调用useCounterStore获取store实例。

```vue

<script setup>
  import { useCounterStore } from '@/stores/counter'
  // 在组件内部的任何地方均可以访问变量 `store` ✨
  const store = useCounterStore()
</script>

```

**从store解构要注意的细节**

useStore方法创建一个被reactive()包裹的store实例，解构store实例会丢失state的响应性， storeToRefs(store)可以解决这个问题，它将为每一个响应式属性创建引用。

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

**(4)变更state**

store.$patch()，传入一个state的补丁对象或者包含操作state的函数来更改store。

**(5)替换state**

你不能完全替换掉 store 的 state，因为那样会破坏其响应性。但是，你可以 patch 它。

```plain

// 这实际上并没有替换`$state`
store.$state = { count: 24 }
// 在它内部调用 `$patch()`：
store.$patch({ count: 24 })

```

你也可以通过变更 `pinia` 实例的 `state` 来设置整个应用的初始 state。这常用于 SSR 中的激活过程。

```plain

pinia.state.value = {}

```

**(6)订阅state**

通过 store 的 `$subscribe()` 方法侦听 state 及其变化。比起普通的 `watch()`，使用 `$subscribe()` 的好处是 subscriptions 在 patch 后只触发一次。

**2.getter**

state的计算值。

**3.action**

state的method。

### 四、对比Vuex
