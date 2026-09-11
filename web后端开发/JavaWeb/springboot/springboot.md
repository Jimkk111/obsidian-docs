---
title: springboot
tags:
  - MOC
---

# springboot

上级：[[web后端开发/JavaWeb/JavaWeb|JavaWeb]]

学习路线：**先入门**（[[web后端开发/JavaWeb/springboot/介绍|介绍]] → [[web后端开发/JavaWeb/springboot/配置文件|配置文件]]）→ **再学 Spring 核心**（spring/ 子目录：IoC/DI 是地基，其他都建立在它之上）→ **最后学 Web 开发**（spring-mvc/ 子目录：接收请求 → 访问数据库 → 拦截器/异常处理/配置）。

持久层框架 MyBatis 本身的完整笔记在 [[web后端开发/数据库/MyBatis/MyBatis|MyBatis]]，数据库基础在 [[web后端开发/数据库/MySql/MySql|MySql]]。

## 本级笔记

- [[web后端开发/JavaWeb/springboot/介绍|介绍]]：Spring 家族、Spring Boot 解决什么问题、快速入门、项目结构约定
- [[web后端开发/JavaWeb/springboot/配置文件|配置文件]]：yml 语法、读取配置（@Value/@ConfigurationProperties）、多环境切换

## 子目录

- [[web后端开发/JavaWeb/springboot/spring/spring|spring]]（Spring 框架核心）
	- [[web后端开发/JavaWeb/springboot/spring/IoC和DI|IoC和DI]]：控制反转与依赖注入，Spring 的地基
	- [[web后端开发/JavaWeb/springboot/spring/Bean管理|Bean管理]]：声明 Bean 的方式、注入方式、作用域、生命周期
	- [[web后端开发/JavaWeb/springboot/spring/注解|注解]]：Spring 常用注解速查表
	- [[web后端开发/JavaWeb/springboot/spring/AOP|AOP]]：面向切面编程，日志/事务的底层原理
	- [[web后端开发/JavaWeb/springboot/spring/事务管理|事务管理]]：@Transactional、传播行为、失效场景
- [[web后端开发/JavaWeb/springboot/spring-mvc/spring-mvc|spring-mvc]]（Web 开发）
	- [[web后端开发/JavaWeb/springboot/spring-mvc/Controller|Controller]]：接收请求（各种参数绑定）、返回 JSON
	- [[web后端开发/JavaWeb/springboot/spring-mvc/Mapper|Mapper]]：持久层、Spring Boot 整合 MyBatis
	- [[web后端开发/JavaWeb/springboot/spring-mvc/拦截器|拦截器]]：HandlerInterceptor、登录校验实战
	- [[web后端开发/JavaWeb/springboot/spring-mvc/异常处理|异常处理]]：全局异常处理、自定义业务异常
	- [[web后端开发/JavaWeb/springboot/spring-mvc/配置|配置]]：WebMvcConfigurer、静态资源、跨域
