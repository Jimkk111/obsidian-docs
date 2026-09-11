---
title: AOP
created: 2026-09-10
updated: 2026-09-10
tags:
  - JavaWeb
---

上级：[[web后端开发/JavaWeb/springboot/spring/spring|spring]]

## 一、AOP 解决什么问题

日志、事务、权限校验、接口耗时统计这类需求有个共同点：**大量业务方法都要，但和业务逻辑无关**。它们叫**横切关注点（Cross-Cutting Concern）**。

不用 AOP 时只能复制粘贴：

```java
public void createOrder(Order order) {
    long start = System.currentTimeMillis();          // 每个方法都要写
    log.info("createOrder 开始执行, 参数: {}", order);  // 每个方法都要写
    // ……真正的业务逻辑……
    log.info("耗时 {}ms", System.currentTimeMillis() - start);
}
```

AOP（Aspect-Oriented Programming，面向切面编程）的做法：把这段公共逻辑抽出来写一次（切面），声明"它要套在哪些方法上"（切点），运行时 Spring 自动把这些逻辑织入目标方法的前后。**业务代码一行不动**。

你早已在用 AOP 的成品：`@Transactional`（事务）就是 AOP 实现的——它给 Service 方法包了一层"开事务→执行→提交/回滚"。

## 二、核心概念

| 术语 | 含义 | 在"统计耗时"例子里对应 |
|---|---|---|
| JoinPoint 连接点 | 程序执行中可以被拦截的点（Spring 限定为**方法执行**） | 每一个方法调用 |
| Pointcut 切点 | 表达式：筛选出要拦截哪些方法 | `service 包下的所有方法` |
| Advice 通知 | 拦截到之后执行的逻辑，以及它在方法前后的位置 | `记开始时间、记结束时间` |
| Aspect 切面 | **切点 + 通知**的组合，一个普通的 @Component 类 | 整个耗时统计类 |
| 织入 | 把切面套到目标方法上的动作（Spring 运行时生成动态代理） | 自动发生 |

一句话：**切面 = 在哪里（切点）+ 做什么（通知）**。

## 三、快速上手

### 1. 引依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### 2. 写切面：统计所有 Service 方法的耗时

```java
@Aspect                // 声明这是切面
@Component             // 切面自己也要是 Bean
@Slf4j
public class TimeLogAspect {

    // 切点表达式：service 包（含子包）下所有类的所有方法
    @Pointcut("execution(* com.example.demo.service..*(..))")
    public void servicePointcut() {}

    @Around("servicePointcut()")          // 环绕通知：方法前后都能插手
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return pjp.proceed();         // 放行，执行目标方法（不调用=方法被拦截）
        } finally {
            String method = pjp.getSignature().toShortString();   // 拿到方法签名
            log.info("{} 耗时 {}ms", method, System.currentTimeMillis() - start);
        }
    }
}
```

### 3. 切点表达式怎么读

```
execution(*  com.example.demo.service..*(..))
          │  │                          │ │ └─ (..) 任意个任意类型参数
          │  │                          │ └─── 方法名任意
          │  │                          └───── .. 该包及其子包
          │  └────────────────────────────── 包名（单级用 . 多级用 ..）
          └───────────────────────────────── 任意返回值类型
```

### 4. 更实用的姿势：注解式切点

按包名拦截太粗暴，常见做法是**自定义一个注解，谁标注解就拦谁**：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TimeLog { }

// 切面里：
@Around("@annotation(com.example.demo.annotation.TimeLog)")
public Object around(ProceedingJoinPoint pjp) throws Throwable { ... }

// 业务方法上：想统计就标注解，不想就不标
@TimeLog
public void createOrder(Order order) { ... }
```

## 四、五种通知类型

```java
@Before("servicePointcut()")               // 目标方法前
public void before(JoinPoint jp) { }

@AfterReturning(value = "servicePointcut()", returning = "result")   // 正常返回后，能拿到返回值
public void afterReturning(JoinPoint jp, Object result) { }

@AfterThrowing(value = "servicePointcut()", throwing = "e")          // 抛异常后，能拿到异常
public void afterThrowing(JoinPoint jp, Exception e) { }

@After("servicePointcut()")                // finally：无论成败都执行
public void after(JoinPoint jp) { }

@Around("servicePointcut()")               // 环绕：功能是前面四个的总和（最常用）
public Object around(ProceedingJoinPoint pjp) throws Throwable { return pjp.proceed(); }
```

`@Around` 一个就能实现所有需求（自己 try-catch-finally 分段），**日常用 @Around 就够了**；其他四个语义更直观，读别人代码要认识。

执行顺序：`@Around 前半 → @Before → 目标方法 → @AfterReturning/@AfterThrowing → @After → @Around 后半`。

## 五、原理与典型应用

Spring AOP 的实现是**运行时动态代理**：容器发现一个 Bean 被切点匹配，注入给你的就不再是原始对象，而是包了一层切面逻辑的**代理对象**。对 JDK 动态代理而言，代理的是接口；没有接口时用 CGLIB 生成子类。

典型应用：

- **声明式事务**：`@Transactional`（见 [[web后端开发/JavaWeb/springboot/spring/事务管理|事务管理]]）
- 统一日志 / 操作审计
- 接口限流、防重复提交
- 权限校验

## ⚠️ 自调用失效

```java
@Service
public class OrderService {

    public void createOrder() {
        this.notify();        // ❌ this 是原始对象，不是代理 → 切面/事务全部失效
    }

    @TimeLog
    public void notify() { }
}
```

切面逻辑在**代理对象**上，而 `this.xxx()` 绕过了代理直接调内部方法。这是 `@Transactional` 最常见的失效原因，解法见 [[web后端开发/JavaWeb/springboot/spring/事务管理|事务管理]] 的失效场景一节。
