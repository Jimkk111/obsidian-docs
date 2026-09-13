---
title: axios
source: https://www.yuque.com/mook-mvqcp/wc9wsu/smf2l1odhaqi8o06
created: 2026-05-02
updated: 2026-05-02
tags:
  - 计算机网络
---

**一、基础用法**

如果想在项目中使用axios，需要先安装axios依赖，然后导入。

使用axios有多种方式 ，可以通过axios(path[,config])（创建的axios也可以用这种方式）来发送请求、或者是axios.*(path[,config])，以下是创建实例的方式。

1.创建axios实例

```javascript

const request = axios.create({
  baseURL:'',
  timeout：,
  headers:
})

```

2.发送请求

axios实例包含get、post、put、update、delete对应的方法，请求属于哪种方法就调用哪种axios对应的方法来发送请求。

有两种方式：

（1）通过实例调用

例如：

```javascript

import request from '@/api/request'

// GET 请求
request.get('/news/list', { params: { page: 1, pageSize: 10 } })
  .then(function (response) {
    console.log(response.data)
  })
  .catch(function (error) {
    console.error('请求失败', error)
  })

// POST 请求
request.post('/auth/login', {
  username: 'admin',
  password: '123456'
})
  .then(function (response) {
    console.log(response.data)
  })

```

(2)用封装好的便捷方法

例如：

```javascript

  import { get, post, put, del, getPaginated } from '@/api/request'

// GET - 获取数据
get('/news/list', { params: { page: 1, pageSize: 10 } })
  .then(function (data) {
    console.log(data) // 已经是解包后的业务数据，不用 .data.data
  })
  .catch(function (error) {
    console.error(error.message)
  })

```

**二、拦截器**

通过axios实例或者是axios的interceptors来设置拦截器

1.请求拦截器

用途：

- 添加通用配置：比如自动在请求头里加上 `Authorization` token、`Content-Type`。

- 数据转换：对提交的参数做统一格式化、加密、或者过滤掉null、undefined字段。

- 请求去重：短时间内相同请求直接拦截不发送。

- 加载状态：开启页面或按钮的loading动画，避免重复点击。

- 日志上报：记录请求的URL、参数、时间等，用于调试或埋点。

语法如下：

```javascript

request.interceptors.request.use(
  成功回调(config),    // 必须返回 config
  失败回调(error)       // 返回 Promise.reject
)

```

2.响应拦截器

用途：

- 统一错误处理：如检测到401未授权，自动跳转到登录页；网络超时或500错误时弹出全局提示。

- 数据结构：后端返回的数据往往包在{data,code,message}里，可以在拦截器里直接取出data返回，业务代码就不用重复判断code是否成功。

- 刷新token：响应返回过期提示时，自动用refresh token请求新token，并重式原始请求。

- 结束loading：无论成功或失败，都关闭对应的loading。

语法如下：

```javascript

request.interceptors.response.use(
  成功回调(response),   // 直接返回 response 或处理后返回
  失败回调(error)        // 统一错误处理
)

```
