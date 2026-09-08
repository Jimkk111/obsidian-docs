---
title: Vue Router
source: https://www.yuque.com/mook-mvqcp/wc9wsu/fozfg5x76gbxtbo9
created: 2026-04-17
updated: 2026-09-08
tags:
  - Vue
---

# Vue Router

> 参考官方文档：[Vue Router 4（Vue3）](https://router.vuejs.org/zh/) / [Vue Router 3（Vue2）](https://v3.router.vuejs.org/zh/)
> 本文以 Vue Router 4（Vue3）写法为主线，Router 3（Vue2）的差异集中标注，完整对照见 [[#十一、Router 3 与 Router 4 差异对照表]]。

上级：[[web前端开发/Vue/Vue|Vue]]

## 目录

1. [[#一、介绍]]
2. [[#二、基础配置与使用]]
3. [[#三、动态路由匹配]]
4. [[#四、嵌套路由]]
5. [[#五、命名路由/命名视图]]
6. [[#六、导航守卫]]
7. [[#七、路由懒加载]]
8. [[#八、路由元信息与滚动行为]]
9. [[#九、动态添加/移除路由]]
10. [[#十、历史模式]]
11. [[#十一、Router 3 与 Router 4 差异对照表]]

---

### 一、介绍

Vue Router是Vue官方的路由管理工具，核心作用是将url与页面组件绑定，实现url更改时自动切换和渲染到对应组件页面，实现无刷新更新页面；其原理是维护一张路径与组件的映射表，监听路径的变化，在 `<router-view>` 处动态渲染对应组件，无刷新切换页。

版本对应：Vue2 → Vue Router 3.x，Vue3 → Vue Router 4.x。

### 二、基础配置与使用

1.从Vue Router导入createRouter和createWebHistory（因历史模式而异）

> Vue2（Router 3）导入的是构造函数：`import VueRouter from 'vue-router'`，并通过 `Vue.use(VueRouter)` 注册插件。

2.使用createRouter创建路由实例

```js
const router = createRouter({
history: ...,
routes:[
// 路径与组件的映射
]
})
```

> Vue2（Router 3）用 `new VueRouter({...})` 创建，历史模式通过 `mode: 'history'` 字符串指定。

3.在main.js中注册路由实例

```js
// Vue3（Router 4）
app.use(router)
```

```js
// Vue2（Router 3）
Vue.use(VueRouter)
new Vue({ router, render: h => h(App) }).$mount('#app')
```

4.设置路由入口`<router-view/>`

视图组件渲染的地方

5.设置路由导航

（1）声明式导航

`<router-link to="path"/>`替代`<a/>`标签，用于无刷新路由跳转。

注意`<router-link/>`用to属性表示目的路径

`router-link-active` 和 `router-link-exact-active`这两个类名可以指定激活样式。

> Router 3 特有：`<router-link>` 支持 `tag` 属性指定渲染成什么标签；Router 4 已移除，一律渲染为 `<a>`。

（2）编程式导航

使用javascript通过router.push(path)或者router.push({name:path})实现路由跳转

```js
// Vue2 选项式组件：this.$router
this.$router.push('/user/1')                            // 字符串路径
this.$router.push({ path: '/user', query: { id: 1 } })  // 带查询参数 /user?id=1
this.$router.push({ name: 'user', params: { id: 1 } })  // 命名路由
this.$router.replace('/login')                          // 不留 history 记录
this.$router.go(-1)                                     // 前进/后退
```

```js
// Vue3 组合式 API：useRouter()
import { useRouter } from 'vue-router'

const router = useRouter()
router.push('/user/1')
router.replace('/login')
router.go(-1)
```

> 注意：`path` 与 `params` 同时使用时 params 会被忽略，需要 params 时用 `name` 或完整字符串路径。

### 三、动态路由匹配

为什么需要动态路由？有些路径与具体的数据有关，比如用户页面的路径，路径中包含用户的id信息，由于每一个用户的id不相同，使用硬编码的路径无法匹配。动态路由的作用就是通过一种动态片段使用一个路由组件去匹配多种具体路径。

**1.基本语法**

在配置routes时，于path中使用冒号:标记动态路由。

```js
const routes = [
  // 动态段叫 id
  { path: '/user/:id', component: User },
  // 一个动态路由可以有多个动态段
  { path: '/post/:postId/comment/:commentId', component: PostComment}
]
```

**2.获取路由参数**

route对象的params属性存储着路由参数，值默认是字符串。

```js
// Vue2 选项式组件
this.$route.params.id
```

```vue
<!-- Vue3 组合式 API：useRoute()（不要解构，会丢失响应性） -->
<script setup>
import { useRoute } from 'vue-router'
const route = useRoute()
console.log(route.params.id)
</script>
```

**3.路由参数发生改变**

路由参数发生变化，该动态路由指向的组件实例会被复用，但有副作用（生命周期函数不生效），这个问题可以通过watch、路由守卫来解决：

```js
// 方式一：watch 参数
// Vue2
watch: { '$route.params.id'(newId) { this.fetchData(newId) } }
// Vue3
watch(() => route.params.id, (newId) => fetchData(newId))

// 方式二：组件内守卫
// Vue2
async beforeRouteUpdate(to, from, next) { await this.fetchData(to.params.id); next() }
// Vue3 组合式 API
onBeforeRouteUpdate(async (to) => { await fetchData(to.params.id) })
```

**4.捕获404页面**

放在路由配置的最末尾，两个版本写法不同：

```js
// Router 4（Vue3）：自定义正则 + 可重复参数
const routes = [
  { path: '/user-:afterUser(.*)', component: UserGeneric },  // /user- 开头的一切
  { path: '/:pathMatch(.*)*', name: 'NotFound', component: NotFound }, // 兜底
]

// Router 3（Vue2）：通配符 *
const routes = [
  { path: '/user-*', component: UserGeneric },
  { path: '*', component: NotFound },   // 必须放最后
]
```

**5.自定义参数正则**

在动态段后面的括号中写正则：

```js
const routes = [
  // /:id 只匹配纯数字（\d+ 需转义为 \\d+）
  { path: '/:id(\\d+)' },
  // 可重复的参数：+ 匹配一个或多个，* 匹配零个或多个
  { path: '/:chapters+' },   // /one/two/three → chapters: ['one','two','three']
]
```

> Router 3 底层同样使用 path-to-regexp，支持该语法，但通配符 `*` 的裸用法仅 Router 3 可用。

**6.嵌套动态路由**

嵌套路由的 children 中同样可以定义动态段，params 会合并到 route.params 中：

```js
const routes = [
  {
    path: '/user/:id',
    component: User,
    children: [
      // /user/123/posts/456 → params: { id: '123', postId: '456' }
      { path: 'posts/:postId', component: UserPost },
    ],
  },
]
```

### 四、嵌套路由

含义：在路由组件的内部包含了自己的`<router-view>`，用于渲染该组件下的子路由。实现了父级页面+子内容区域路由的结构。这有利于减少重复代码。

**1.基本语法**

在定义单个路由时指定这个路由的children，形成路由的嵌套结构（允许任意层级）

```js
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

2.props传参或者通过useRoute获取父级参数

> props 传参：路由配置 `props: true` 可将 `route.params` 作为 props 传给组件，使组件与路由解耦（两个版本均支持）。

### 五、命名路由/命名视图

**1.命名路由**

给路由起一个名字，在用到路径的地方使用这个名字替代。

```js
const routes = [
  { path: '/user/:id', name: 'user', component: User }
]
```

```html
<router-link :to="{ name: 'user', params: { id: 123 } }">用户123</router-link>
```

```js
// Vue2: this.$router.push(...)；Vue3 组合式: router.push(...)
router.push({ name: 'user', params: { id: 123 } })
```

**2.命名视图**

命名视图 是指：给 `<router-view>` 起一个名字（`name` 属性），然后在路由配置中通过 `components` 字段（注意是复数 `components`）为每个命名视图指定要渲染的组件。

把一个路由对应的组件拆分为多个子组件，实现灵活组合页面。

```html
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

```js
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

```js
// Router 4（Vue3）：next 可选，支持返回值——
// return false 取消导航；return 路由地址重定向；不返回则放行
router.beforeEach((to, from) => {
  if (to.name !== 'Login' && !isAuthenticated) return { name: 'Login' }
})

// Router 3（Vue2）：必须调用 next()——next() 放行 / next(false) 取消 / next('/login') 重定向
router.beforeEach((to, from, next) => {
  if (to.name !== 'Login' && !isAuthenticated) next({ name: 'Login' })
  else next()
})

// 两个版本共有：
router.beforeResolve((to, from) => {})   // 导航被确认之前、异步组件解析后
router.afterEach((to, from, failure) => {}) // 导航完成后（不能改变导航）
```

2. **路由独享守卫**：仅作用于某个特定路由。

在路由配置中直接定义 `beforeEnter` 守卫，只对该路由生效。

```js
const routes = [
  {
    path: '/admin',
    component: AdminPanel,
    beforeEnter: (to, from, next) => {
      if (!store.state.user.isAdmin) {
        next('/forbidden')     // Router 4 中也可直接 return '/forbidden'
      } else {
        next()
      }
    }
  }
]
```

3. **组件内守卫**：作用于某个组件内部。

在路由组件中直接定义，与组件生命周期钩子类似。

```js
// Router 3 / Router 4 选项式组件均可使用：
export default {
  // 渲染该组件的路由被 confirm 前调用；实例还未创建，拿不到 this
  beforeRouteEnter(to, from, next) {
    next(vm => { /* 通过 vm 访问组件实例 */ })
  },
  // 当前路由改变、组件被复用时调用
  beforeRouteUpdate(to, from, next) {},
  // 离开该组件对应的路由时调用
  beforeRouteLeave(to, from, next) {},
}
```

```vue
<!-- Router 4（Vue3）组合式 API 另外提供： -->
<script setup>
import { onBeforeRouteUpdate, onBeforeRouteLeave } from 'vue-router'

onBeforeRouteUpdate(async (to) => { /* 参数变化时 */ })
onBeforeRouteLeave((to, from) => {
  const answer = window.confirm('还有未保存的数据，确定离开吗？')
  if (!answer) return false
})
</script>
```

**导航解析流程**：导航被触发 → 失活组件 `beforeRouteLeave` → 全局 `beforeEach` → 复用组件 `beforeRouteUpdate` → 路由 `beforeEnter` → 异步路由组件解析 → 组件内 `beforeRouteEnter` → 全局 `beforeResolve` → 导航被确认 → 全局 `afterEach` → DOM 更新 → `beforeRouteEnter` 的 next 回调。

2.导航守卫的应用

- 权限控制
- 页面拦截
- 数据预取
- 日志埋点
- 动态修改页面标题

### 七、路由懒加载

在切换到目标路由时才加载对应组件。

```js
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

> Router 3（Vue2）写法相同，`new VueRouter({ routes })`；webpack 环境可用注释分组：`() => import(/* webpackChunkName: "group-user" */ './User.vue')`。

### 八、路由元信息与滚动行为

**路由元信息（meta）**：路由配置的 meta 字段可附加自定义数据，常配合守卫做权限校验：

```js
const routes = [
  {
    path: '/admin',
    component: Admin,
    meta: { requiresAuth: true, title: '后台管理' },
  },
]
```

**滚动行为（scrollBehavior）**：切换路由时控制页面滚动位置（仅 history 模式支持）：

```js
// Router 4
const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) return savedPosition       // 前进后退还原位置
    if (to.hash) return { el: to.hash }           // 锚点定位
    return { top: 0 }                             // 总是滚动到顶部
  },
})

// Router 3：位置写法为 { x: 0, y: 0 }，锚点用 { selector: to.hash }
```

### 九、动态添加/移除路由

运行时动态添加 / 移除路由（常用于权限菜单），两个版本 API 不同：

```js
// Router 4：单个添加/移除
router.addRoute({ path: '/new', component: NewComp })
router.addRoute('parentName', routeRecord)   // 添加为某路由的子路由
router.removeRoute('name')
router.hasRoute('name')
router.getRoutes()

// Router 3：只有批量添加，无法移除
router.addRoutes([routeRecord, ...])   // 已废弃
```

### 十、历史模式

| 模式 | Router 4（Vue3） | Router 3（Vue2） | URL 形式 | 说明 |
| --- | --- | --- | --- | --- |
| HTML5 模式 | `createWebHistory()` | `mode: 'history'` | `/user/1`（无 #） | 推荐，需服务器配置 fallback 到 index.html |
| Hash 模式 | `createWebHashHistory()` | `mode: 'hash'`（默认） | `/#/user/1` | 无需服务器配置，SEO 较差 |
| Memory 模式 | `createMemoryHistory()` | `mode: 'abstract'` | 无 URL | SSR / 测试环境使用 |

### 十一、Router 3 与 Router 4 差异对照表

| | Vue Router 3（Vue2） | Vue Router 4（Vue3） |
| --- | --- | --- |
| 创建路由 | `new Router({...})` + `Vue.use(VueRouter)` | `createRouter({...})` + `app.use(router)` |
| 历史模式 | `mode: 'history'` | `history: createWebHistory()` |
| 组件内访问 | `this.$router` / `this.$route` | 同左，另增 `useRouter()` / `useRoute()` |
| 守卫放行方式 | 必须调用 `next()` | `next` 可选，支持返回值（false / 路由地址） |
| 404 匹配 | `path: '*'` 通配符 | `/:pathMatch(.*)*` 自定义正则 |
| 动态添加路由 | `router.addRoutes(routes)`（批量，已废弃） | `router.addRoute()` / `router.removeRoute()`（单个） |
| router-link | 支持 `tag`、`event` 属性 | 一律渲染 `<a>`，移除 `tag` / `event` |
| scrollBehavior 位置 | `{ x: 0, y: 0 }` | `{ top: 0 }` |

## 相关笔记

- [[web前端开发/Vue/Vue2教程|Vue2教程]]
- [[web前端开发/Vue/Vue3教程|Vue3教程]]
- [[web前端开发/Vue/Pinia|Pinia]]
