---
title: spring-mvc
tags:
  - MOC
---

# spring-mvc

上级：[[web后端开发/JavaWeb/springboot/springboot|springboot]]

Web 开发层。先理解下面两个全局认知，再按顺序学笔记。

## 三层架构与 MVC

**三层架构**是后端代码的组织方式，依赖只能单向向下：

```
Controller（表现层）  接收请求、校验参数、返回响应，不写业务逻辑
    ↓
Service（业务层）     业务逻辑、事务控制，一个方法对应一个业务动作
    ↓
Mapper（持久层）      只做数据库读写，一个方法对应一条 SQL
```

**MVC** 是表现层内部的分工：Model（数据模型）、View（视图）、Controller（控制器）。前后端分离后，View 交给前端框架（Vue 等），后端只剩 Controller + Model，以 JSON 交互——所以日常写的就是「Controller 接参 → 调 Service → 返回 Result」。

## 一次请求的完整流程

```
浏览器发请求
  → DispatcherServlet（前端控制器，一切请求的入口）
  → HandlerMapping（根据路径找到对应的 Controller 方法 + 拦截器链）
  → 拦截器 preHandle
  → HandlerAdapter（解析参数、调用方法）
  → Controller → Service → Mapper → 返回结果
  → HttpMessageConverter（把返回值序列化成 JSON）
  → 拦截器 postHandle / afterCompletion
  → 浏览器拿到 JSON
```

记住 DispatcherServlet 是总入口即可，中间环节 Spring 自动完成。理解这个流程，拦截器、异常处理、参数解析在哪个环节生效就都清楚了。

## 笔记

- [[web后端开发/JavaWeb/springboot/spring-mvc/Controller|Controller]]：@RestController、参数接收（@RequestParam/@PathVariable/@RequestBody）、统一响应
- [[web后端开发/JavaWeb/springboot/spring-mvc/Mapper|Mapper]]：持久层、Spring Boot 整合 MyBatis（@Mapper、XML/注解 SQL）
- [[web后端开发/JavaWeb/springboot/spring-mvc/拦截器|拦截器]]：HandlerInterceptor 三个方法、登录校验实战、ThreadLocal 传递用户
- [[web后端开发/JavaWeb/springboot/spring-mvc/异常处理|异常处理]]：@RestControllerAdvice 全局兜底、自定义业务异常
- [[web后端开发/JavaWeb/springboot/spring-mvc/配置|配置]]：WebMvcConfigurer（推荐）vs WebMvcConfigurationSupport（慎用）、静态资源、跨域
