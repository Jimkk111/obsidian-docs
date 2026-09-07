---
title: Vue Router
source: https://www.yuque.com/mook-mvqcp/wc9wsu/fozfg5x76gbxtbo9
created: 2026-04-17
updated: 2026-04-17
tags:
  - Vue
---

### 一、介绍

Vue Router是Vue官方的路由管理工具，核心作用是将url与页面组件绑定，实现url更改时自动切换和渲染到对应组件页面，实现无刷新更新页面；其原理是维护一张路径与组件的映射表，监听路径的变化，在 `<router-view>` 处动态渲染对应组件，无刷新切换页。

### 二、基础配置与使用

1.从Vue Router导入createRouter和createWebHistory（因历史模式而异）

2.使用createRouter创建路由实例

```javascript

const router = createRouter({
history: ...,
routes:[
// 路径与组件的映射
]
})

```

3.在main.js中注册路由实例

```javascript

Vue.use(router)

```

4.设置路由入口<router-view/>

视图组件渲染的地方

5.设置路由导航

（1）声明式导航

<router-link to="path"/>替代<a/>标签，用于无刷新路由跳转。

注意<router-link/>用to属性表示目的路径

`router-link-active` 和 `router-link-exact-act`这两个类名可以指定激活样式。

（2）编程式导航

使用javascript通过router.push(path)或者router.push({name:path})实现路由跳转

### 三、动态路由匹配

为什么需要动态路由？有些路径与具体的数据有关，比如用户页面的路径，路径中包含用户的id信息，由于每一个用户的id不相同，使用硬编码的路径无法匹配。动态路由的作用就是通过一种动态片段使用一个路由组件去匹配多种具体路径。

**1.基本语法**

在配置routes时，于path中使用冒号:标记动态路由。

```javascript

const routes = [
  // 动态段叫 id
  { path: '/user/:id', component: User },
  // 一个动态路由可以有多个动态段
  { path: '/post/:postId/comment/:commentId', component: PostComment}
]

```

**2.获取路由参数**

route对象的parms属性存储着路由参数，值默认是字符串。

**3.路由参数发生改变**

路由参数发生变化，该动态路由指向的组件实例会被复用，但有副作用（生命周期函数不生效），这个问题可以通过watch、路由守卫来解决。

**4.捕获404页面**

**5.自定义参数正则**

**6.嵌套动态路由**

### 四、嵌套路由

含义：在路由组件的内部包含了自己的<router-view>，用于渲染该组件下的子路由。实现了父级页面+子内容区域路由的结构。这有利于减少重复代码。

**1.基本语法**

在定义单个路由时指定这个路由的children，形成路由的嵌套结构（允许任意层级）

```javascript

const routes = [
  {
    path: '/',
    component: fathercomponent,        // 父组件
    children: [               // 子路由
      {
        path: '',             // 空路径 -> 默认子路由
        name: 'son1',
        component: son2component
      },
      {
        path: 'son2',       // 子路由路径会自动拼接父路径：/orders
        name: 'son2',				// 子路由路径以/开头会覆盖父路径，破坏嵌套结构的语义
        component: son2component
      }
    ]
  }
]

```

**2.参数传递与继承**

1.在子路由加上参数

2.pros传参或者通过useRoute获取父级参数

### 五、命名路由/命名视图

**1.命名路由**

给路由起一个名字，在用到路径的地方使用这个名字替代。

```javascript

const routes = [
  { path: '/user/:id', name: 'user', component: User }
]

```

```vue

<RouterLink :to="{ name: 'user', params: { id: 123 } }">用户123</RouterLink>

```

```vue

router.push({ name: 'user', params: { id: 123 } })

```

**2.命名视图**

命名视图 是指：给 `<router-view>` 起一个名字（`name` 属性），然后在路由配置中通过 `components` 字段（注意是复数 `components`）为每个命名视图指定要渲染的组件。

把一个路由对应的组件拆分为多个子组件，实现灵活组合页面。

```vue

<!-- App.vue 或某个布局组件 -->
<template>
  <div>
    <!-- 顶部导航区 -->
    <router-view name="header"></router-view>
    <!-- 侧边栏区 -->
    <router-view name="sidebar"></router-view>
    <!-- 主内容区（默认视图） -->
    <router-view></router-view>
  </div>
</template>

```

```vue

// router/index.js
import HeaderView from '@/views/Header.vue'
import SidebarView from '@/views/Sidebar.vue'
import MainContentView from '@/views/MainContent.vue'

const routes = [
  {
    path: '/dashboard',
    components: {
      default: MainContentView,    // 没有名字的 <router-view> 对应 default
      header: HeaderView,
      sidebar: SidebarView
    }
  }
]

```

### 六、导航守卫

导航守卫 是 Vue Router 提供的一系列钩子函数，它们在路由跳转的不同阶段被调用，允许你在路由进入、离开或更新时执行自定义逻辑（如检查登录状态、确认未保存的更改、获取异步数据等）。

1.守卫分类

1. **全局守卫**：作用于所有路由跳转。

全局守卫是在创建 `router` 实例时挂载的钩子，会影响每一次路由跳转。

1. **路由独享守卫**：仅作用于某个特定路由。

在路由配置中直接定义 `beforeEnter` 守卫，只对该路由生效。

```vue

const routes = [
  {
    path: '/admin',
    component: AdminPanel,
    beforeEnter: (to, from, next) => {
      if (!store.state.user.isAdmin) {
        next('/forbidden')
      } else {
        next()
      }
    }
  }
]

```

1. **组件内守卫**：作用于某个组件内部。

在路由组件中直接定义，与组件生命周期钩子类似。

2.导航守卫的应用

- 权限控制

- 页面拦截

- 数据预取

- 日志埋点

- 动态修改页面标题

### 七、路由懒加载

在切换到目标路由时才加载对应组件。

```vue

import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'

const routes = [
  {
    path: '/about',
    name: 'about',
    // 路由懒加载：只有访问该路径时，才会加载对应的组件代码块
    component: () => import('../views/AboutView.vue')
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router

```
