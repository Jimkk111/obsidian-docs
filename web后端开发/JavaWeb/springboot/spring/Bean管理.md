---
title: Bean管理
created: 2026-09-10
updated: 2026-09-10
tags:
  - JavaWeb
---

上级：[[web后端开发/JavaWeb/springboot/spring/spring|spring]]

前置：[[web后端开发/JavaWeb/springboot/spring/IoC和DI|IoC和DI]]（容器怎么工作的），本篇讲怎么把类放进容器、怎么拿出来用、放进容器之后对象的一生。

## 一、什么是 Bean

被 Spring IoC 容器创建和管理的对象就是 Bean。**容器里放的是"对象实例"，不是类**——默认每个 Bean 在容器里只有一个实例（单例），所有注入方共享同一个。

## 二、声明 Bean 的两种主流方式

### 1. 注解扫描（自己写的类）

类上加 `@Component` 或它的语义化衍生注解，启动时被 `@ComponentScan` 扫到后自动注册：

| 注解 | 语义 | 放在哪层 |
|---|---|---|
| `@Component` | 通用组件 | 不属于下面三层的 |
| `@Service` | 业务逻辑 | Service 层 |
| `@Repository` | 数据访问 | Mapper/Dao 层（还附带持久层异常转换） |
| `@Controller` / `@RestController` | 接收请求 | Controller 层 |

四者对容器而言**完全等价**（源码上都是 `@Component` 的派生），分开写纯粹是让读代码的人一眼看出这个类在哪一层。默认 Bean 名 = 类名首字母小写（`UserService` → `userService`）。

### 2. @Bean 方法（第三方 jar 里的类）

第三方类你改不了源码、加不了 `@Component`，用配置类注册：

```java
@Configuration
public class WebConfig {

    @Bean                      // 返回值进容器，方法名默认就是 Bean 名
    public RestTemplate restTemplate() {
        RestTemplate template = new RestTemplate();
        // 需要的定制……
        return template;
    }
}
```

记忆口诀：**自己的类上加注解，别人的类写 @Bean 方法**。

> 了解：还有 `@Import`（导入配置类）和 `@Conditional` 系列条件装配（满足条件才注册），Spring Boot 自动配置就是靠 `@ConditionalOnClass`、`@ConditionalOnMissingBean` 这类注解实现的。

## 三、注入 Bean 的三种方式

```java
// 方式一：构造器注入（官方推荐）
@Service
public class OrderService {
    private final UserService userService;    // 可以 final，对象不可变

    public OrderService(UserService userService) {   // Spring 4.3+ 单构造器不用写 @Autowired
        this.userService = userService;
    }
}

// 配合 Lombok 连构造器都不用写：
@Service
@RequiredArgsConstructor      // 为所有 final 字段生成构造器
public class OrderService {
    private final UserService userService;
}

// 方式二：Setter 注入（可选依赖场景，基本不用）
@Service
public class OrderService {
    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}

// 方式三：字段注入（最省事，但不推荐）
@Service
public class OrderService {
    @Autowired                // 反射直接塞值
    private UserService userService;
}
```

| | 构造器注入 | Setter 注入 | 字段注入 |
|---|---|---|---|
| 推荐 | ✅ 官方推荐 | 可选依赖 | ❌ 图省事 |
| final 修饰 | 可以 | 不行 | 不行 |
| 脱离容器测试 | 直接 new 传 mock | 可以 | 很别扭（要反射塞） |
| 循环依赖 | 启动直接报错（暴露设计问题） | 能靠缓存解 | 能靠缓存解 |

字段注入的问题：字段不能 final、离开 Spring 容器就无法组装、隐藏了"这个类到底依赖了多少东西"。**养成习惯：一律 `@RequiredArgsConstructor` + final 字段**。

## 四、同一类型有多个 Bean 怎么办

接口 `UserService` 有 `UserServiceImplA`、`UserServiceImplB` 两个实现时，注入必然歧义，Spring 启动报 `NoUniqueBeanDefinitionException`。三种解法：

```java
// 1. @Primary：在某个实现类上标"默认选我"，其余照常按类型注入
@Primary
@Service
public class UserServiceImplA implements UserService { ... }

// 2. @Qualifier：注入时点名要哪个
public OrderService(@Qualifier("userServiceImplB") UserService userService) { ... }

// 3. @Resource：按名称注入（jakarta.annotation，JDK 标准）
@Resource(name = "userServiceImplB")
private UserService userService;
```

`@Autowired` vs `@Resource`：`@Autowired` 是 Spring 的，**先按类型**匹配，多个再按字段名/`@Qualifier` 消歧；`@Resource` 是 JDK 标准，**先按名称**匹配，找不到再按类型。日常用 `@Autowired` 就够，遇到按名字区分的场景用 `@Resource` 或 `@Qualifier`。

还有一个技巧：注入集合类型会拿到**所有实现**，策略模式常用：

```java
public OrderService(Map<String, PayService> payServices) { }   // key=Bean名, value=每个实现
```

## 五、作用域 @Scope

```java
@Service
@Scope("prototype")        // 默认 singleton
public class ReportGenerator { ... }
```

| 作用域                                   | 含义                         |
| ------------------------------------- | -------------------------- |
| `singleton`                           | **默认**。容器中只有一个实例，大家共享      |
| `prototype`                           | 每次注入/获取都 new 一个新实例，销毁不由容器管 |
| `request` / `session` / `application` | Web 环境下每个请求/会话一个实例         |
|                                       |                            |

⚠️ 单例 Bean 的成员变量会被并发共享，**不要在单例里存"当前请求的用户"这类会话状态**（那是 ThreadLocal 的事，见 [[web后端开发/JavaWeb/springboot/spring-mvc/拦截器|拦截器]]），否则并发下互相覆盖。

## 六、生命周期

简化版链路，够日常使用：

```
实例化（构造器）
  → 依赖注入（@Autowired 的字段/构造参数被填上）
  → 初始化回调 @PostConstruct
  → 就绪，被使用
  → 容器关闭 → 销毁回调 @PreDestroy
```

```java
@Component
public class CacheWarmer {

    @PostConstruct          // 依赖注入完成后自动调用，适合做初始化（预热缓存、校验配置）
    public void init() {
        System.out.println("缓存预热……");
    }

    @PreDestroy             // 容器关闭（应用正常退出）前调用，适合释放资源
    public void cleanup() {
        System.out.println("释放连接……");
    }
}
```

完整顺序（了解即可，面试会问）：实例化 → 属性填充 → `Aware` 接口回调 → `BeanPostProcessor` 前置处理 → `@PostConstruct` → `InitializingBean.afterPropertiesSet()` → 自定义 `init-method` → `BeanPostProcessor` 后置处理（**AOP 代理一般在这生成**） → 使用 → `@PreDestroy` → `DisposableBean.destroy()` → 自定义 `destroy-method`。

⚠️ prototype Bean 的销毁回调**不会**被容器调用——容器只负责创建它，不管回收。

## ⚠️ 循环依赖

A 构造器依赖 B，B 又构造器依赖 A → 启动直接报错（谁也没法先创建完）。

- Spring 用**三级缓存**解决了"单例 + setter/字段注入"的循环依赖（提前暴露半成品对象）
- 但 Spring Boot 2.6+ **默认全面禁止循环依赖**，有循环直接报错

出现循环依赖先别想着开开关，**它几乎总是设计问题的信号**：通常该把两边的公共逻辑抽到第三个类里，让依赖重新变成单向。

## 关联

- 注解速查 → [[web后端开发/JavaWeb/springboot/spring/注解|注解]]
- AOP 代理与 Bean 的关系 → [[web后端开发/JavaWeb/springboot/spring/AOP|AOP]]
