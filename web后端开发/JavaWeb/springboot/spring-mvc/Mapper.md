---
title: Mapper
source: https://www.yuque.com/mook-mvqcp/viz8gx/qk6exb2pmm44yq6v
created: 2026-09-05
updated: 2026-09-10
tags:
  - JavaWeb
---

上级：[[web后端开发/JavaWeb/springboot/spring-mvc/spring-mvc|spring-mvc]]

持久层（三层架构最底层）：只负责数据库读写，一个方法对应一条 SQL。本篇记 **Spring Boot 整合 MyBatis**；MyBatis 本身（动态 SQL、缓存等）的完整笔记在 [[web后端开发/数据库/MyBatis/MyBatis|MyBatis]]，SQL 语言在 [[web后端开发/数据库/MySql/MySql|MySql]]。

## 一、整合步骤

### 1. 引依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.x</version>   <!-- Spring Boot 3 用 3.x；2.x 用 2.x -->
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

### 2. 写配置

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: 123456

mybatis:
  mapper-locations: classpath:mapper/*.xml      # XML 方式写 SQL 的文件位置
  configuration:
    map-underscore-to-camel-case: true          # 数据库 create_time → 实体 createTime
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl   # 开发期控制台打印 SQL
```

### 3. 写 Mapper 接口 + 注册

```java
@Mapper                                    // 关键：让 MyBatis 生成实现并交给容器
public interface UserMapper {

    @Select("select * from user where id = #{id}")
    User getById(Long id);

    @Insert("insert into user(name, age) values(#{name}, #{age})")
    void insert(User user);
}
```

### 4. Service 注入使用

```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserMapper userMapper;   // 和普通 Bean 一样注入
}
```

## 二、@Mapper 与 @MapperScan

给每个接口写 `@Mapper` 很烦，可以在启动类（或配置类）上批量扫描：

```java
@MapperScan("com.example.demo.mapper")     // 该包下所有接口自动注册为 Mapper
@SpringBootApplication
public class DemoApplication { ... }
```

二选一即可。原理：MyBatis 为接口生成**动态代理实现类**（这就是为什么只写接口不写实现类，方法就能直接跑 SQL）。

## 三、SQL 的两种写法

### 1. 注解方式：简单 SQL

```java
@Select("select * from user where id = #{id}")
@Update("update user set name = #{name} where id = #{id}")
@Delete("delete from user where id = #{id}")
```

### 2. XML 方式：复杂 SQL

接口只声明方法：

```java
List<User> list(@Param("name") String name, @Param("minAge") Integer minAge);
```

`resources/mapper/UserMapper.xml`（namespace 必须是接口全限定名，id 必须是方法名）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.demo.mapper.UserMapper">

    <select id="list" resultType="com.example.demo.entity.User">
        select * from user
        <where>
            <if test="name != null and name != ''">
                and name like concat('%', #{name}, '%')
            </if>
            <if test="minAge != null">
                and age &gt;= #{minAge}
            </if>
        </where>
    </select>
</mapper>
```

**选择标准**：单表简单 SQL 用注解；带 `<if>`/`<foreach>` 动态拼接的复杂 SQL 用 XML。

## 四、参数传递与 #{} / ${}

```java
// 单个参数：直接用
@Select("select * from user where id = #{id}")
User getById(Long id);

// 多个参数：@Param 起名字，SQL 里按名字引用
@Select("select * from user where name = #{name} and age = #{age}")
User find(@Param("name") String name, @Param("age") Integer age);

// 对象参数：直接引用属性名
@Insert("insert into user(name, age) values(#{name}, #{age})")
void insert(User user);
```

| | `#{}` | `${}` |
|---|---|---|
| 原理 | **预编译**占位符 `?`，值后填入 | 字符串**直接拼接**进 SQL |
| 安全 | ✅ 防 SQL 注入 | ❌ 有注入风险 |
| 场景 | **传值一律用它** | 只能传值的场景排除后，如 `order by ${column}` 动态列名 |

## 五、完整链路示例

```java
// Controller → Service → Mapper
@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @GetMapping("/{id}")
    public Result<User> getById(@PathVariable Long id) {
        return Result.success(userService.getById(id));
    }
}

@Service
@RequiredArgsConstructor
public class UserService {
    private final UserMapper userMapper;

    public User getById(Long id) {
        return userMapper.getById(id);       // 查不到是 null，业务上要抛"不存在"
    }
}
```

事务加在 Service 方法上：`@Transactional`，见 [[web后端开发/JavaWeb/springboot/spring/事务管理|事务管理]]。

## ⚠️ 常见坑

| 报错/现象 | 原因 |
|---|---|
| `Invalid bound statement (not found)` | 没加 `@Mapper`/`@MapperScan`；XML 的 namespace/id 与接口不匹配；`mapper-locations` 路径写错 |
| 查出来字段是 null | 没开 `map-underscore-to-camel-case`，下划线列没映射到驼峰属性 |
| 看到 SQL 执行结果 | `log-impl` 配置打印 SQL，或日志级别调 debug |
| 想确认代理生成 | MyBatis 动态代理：接口无需实现类，注入即用 |
