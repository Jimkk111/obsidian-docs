---
title: IoC和DI
source: https://www.yuque.com/mook-mvqcp/viz8gx/dlz3fr9oumwcigk6
created: 2026-08-16
updated: 2026-09-05
tags:
  - JavaWeb
---

上级：[[web后端开发/JavaWeb/springboot/spring/spring|spring]]

## 一、IoC 是什么

**IoC（Inversion of Control，控制反转）**：把对象的创建和依赖管理的控制权，从程序代码本身反转给外部容器。

传统写法：类需要什么就自己 `new`，控制权在自己手里；IoC 写法：只声明"我需要什么"（通常依赖抽象接口），由容器负责创建、组装、注入。

**DI（Dependency Injection，依赖注入）** 是实现 IoC 最常用的手段：IoC 是思想，DI 是落地方式。

"控制反转"的落点：**依赖怎么构造出来这个控制权，从使用方反转到了容器手里**——使用方只拿到一个"已建好、能用"的实例，依赖内部怎么建的对它完全不可见。

## 二、解决什么问题 / 好处

传统 `private MySqlUserRepository repo = new MySqlUserRepository();` 的问题：

1. **强耦合**：类和具体实现绑死，换实现要改所有 `new` 的地方
2. **传递性依赖知识污染**：`new MySqlUserRepository(pool)` 意味着使用方还得懂它的构造参数怎么来（pool 哪来、config 哪来……），依赖类改构造函数，所有使用方跟着编译报错
3. **难以单测**：依赖藏在内部，没法换成 mock
4. **装配逻辑散落**：整棵依赖树每个使用方都要重复搭一遍，共享实例（连接池等）没法统一管理

用 IoC 后，三类收益：

- **换实现 = 改一条绑定**（接口、业务类一行不动）
- **可测试**：构造函数注入一个假实现即可，全程不碰数据库
- **组装责任集中**：只有容器搭一次依赖树

⚠️ 可测试性其实来自"依赖通过构造函数从外面传"这个写法，容器只是把生产环境下的注入自动化了。**小项目、脚本、不写测试的场景直接 `new` 更直白，别为了用而用。**

## 三、IoC 容器的核心原理（手搓最小版）

容器核心就三件事：**注册**（接口→实现的映射表）、**解析**（查表+递归实例化）、**注入**（把整棵依赖树自动装好）。

```java
class Container {
    // 注册表：接口/抽象 -> 具体实现类
    private final Map<Class<?>, Class<?>> bindings = new HashMap<>();
    // 单例缓存
    private final Map<Class<?>, Object> singletons = new HashMap<>();

    public <T> void bind(Class<T> abstraction, Class<? extends T> impl) {
        bindings.put(abstraction, impl);
    }

    @SuppressWarnings("unchecked")
    public <T> T resolve(Class<T> type) throws Exception {
        // 1. 单例缓存命中直接返回
        if (singletons.containsKey(type)) return (T) singletons.get(type);

        // 2. 查注册表；没注册过就把它自己当实现类（具体类天然可实例化）
        Class<?> impl = bindings.getOrDefault(type, type);

        // 3. 反射拿构造器，递归解析每个参数类型（依赖的依赖自动装配）
        Constructor<?> ctor = impl.getDeclaredConstructors()[0];
        Object[] args = new Object[ctor.getParameterCount()];
        for (int i = 0; i < args.length; i++) {
            args[i] = resolve(ctor.getParameterTypes()[i]);
        }

        // 4. 反射创建实例，缓存后返回
        Object instance = ctor.newInstance(args);
        singletons.put(type, instance);
        return (T) instance;
    }
}
```

`resolve()` 的四步就是全部原理：**查缓存 → 查表（找不到就用自己）→ 递归解析构造参数 → 反射实例化并缓存**。真实框架只是在这个骨架上加特性。

使用：

```java
Container c = new Container();
c.bind(UserRepository.class, MySqlUserRepository.class); // 注册：抽象 -> 实现

OrderService svc = c.resolve(OrderService.class);        // 容器递归装配整棵依赖树
```

要点：

- **只有"接口→实现"才需要显式注册**，具体类没注册时 `getOrDefault(type, type)` 回退为用自己，所以业务类不必逐个 bind
- 递归能工作的前提也在这里：`resolve(OrderService.class)` → 递归 `resolve(UserRepository.class)` → 查表得 `MySqlUserRepository`
- 忘了注册接口时会走"用自己"分支，反射实例化接口会抛 `InstantiationException`——真实框架会先检查，抛 "No qualifying bean of type XXX" 这种友好错误

### 1. 多个构造器怎么办

`getDeclaredConstructors()[0]` 是偷懒写法（JDK 不保证数组顺序），真实容器的消歧规则（Spring）：

1. 只有一个构造器 → 直接用它
2. 多个 → 选标了 `@Autowired` 的那个
3. 多个且没标注解 → 用无参构造器；没有就启动报错

Spring 4.3+ 推荐：**每个类只写一个构造器**，既无歧义又不用写注解。

### 2. 为什么用 `ctor.newInstance()`，不能直接 `new`？

**`new` 后面必须跟编译期写死的类名，而 `impl` 是运行时才从 Map 里查出来的变量**，通用代码里根本写不出任何具体的 `new Xxx()`。

硬用 `new` 只能写成 if-else 枚举每个类（每加一个类回来改一处，还丢掉了递归装配），正好毁掉容器"新增类不用改装配"的使命。

| | `new MySqlUserRepository(pool)` | `ctor.newInstance(args)` |
|---|---|---|
| 用哪个类 | 编译期写死 | `impl` 变量，运行时决定 |
| 参数 | 手写、编译期检查 | `Object[]`，运行时匹配 |
| 类型错误 | 编译不过 | 运行时抛异常 |

`ctor.newInstance(args)` 与 `new` 做的事完全一样（分配对象、初始化、返回引用），代价是反射慢一点、失去编译期检查，换来**一段代码处理无限多的类**——这是容器"你加类我不改"的前提。

反射细节：私有构造器要先 `ctor.setAccessible(true)`；构造器内部抛异常会被包成 `InvocationTargetException`，需拆包拿真实异常。

## 四、使用 IoC 容器（Spring）的步骤

Spring 把手搓版的 `bind()` 也省了，通过注解扫描自动注册：

```java
@Component                                  // 1. 标注：这个类交给容器管（自动注册）
class MySqlUserRepository implements UserRepository { ... }

@Service
class OrderService {
    private final UserRepository repo;
    OrderService(UserRepository repo) {     // 2. 构造器注入：只声明依赖，Spring 4.3+ 单构造器不用写 @Autowired
        this.repo = repo;
    }
}
```

步骤总结：

1. 类上加 `@Component`/`@Service` 等注解，声明为 Bean（等价于自动 bind）
2. 启动时容器扫描注解，建立注册表
3. 依赖通过构造器参数声明，容器递归解析并注入（等价于自动 resolve）
4. 入口处从容器取对象（如 `SpringApplication.run()` 返回的 `ApplicationContext.getBean()`），日常开发中连这步都不用，直接注入

在此基础上，Spring 还补了：Bean 生命周期回调（`@PostConstruct` 等）、作用域（singleton 默认 / prototype 等）、循环依赖处理（三级缓存，仅限单例 + setter/字段注入）、AOP 代理（事务、日志）、条件装配（profile）。

## 五、相关注解

1. `@Component`（及衍生的 `@Service`、`@Repository`、`@Controller`）：声明一个类是 Bean，交给容器管理
2. `@Autowired`：自动注入依赖，引入 Bean 时使用；构造器注入时单构造器可省略
3. `@Qualifier` / `@Primary`：同一接口有多个实现时，指定注入哪一个
4. `@Data`：是 Lombok 的注解，自动生成 getter/setter 等，**与声明 Bean 无关**（易混点）

## 六、一句话总结

每个类只**声明**依赖抽象（构造器参数），创建和组装整棵依赖树的责任**移交容器**（查表 + 反射递归 + `newInstance`）。省去 `new` 是机制不是目的，真正买到的是：换实现改一条绑定、依赖可注入 mock 独立测试、装配逻辑集中且共享实例天然单例。
