---
title: Java客户端
source: https://docs.spring.io/spring-data/redis/reference/
created: 2026-09-10
updated: 2026-09-10
tags:
  - 数据库
---

# Java 客户端

## 主流客户端

| 客户端 | 特点 |
| ---- | ---- |
| Jedis | 简单直接；实例**非线程安全**，需配合连接池使用 |
| Lettuce | 基于 Netty；连接实例**线程安全**；**Spring Boot 默认** |
| Redisson | 分布式框架：分布式锁、分布式集合等高级对象 → [[web后端开发/数据库/Redis/分布式锁|分布式锁]] |

**Spring Data Redis** 是 Spring 对 Redis 的封装，底层默认使用 Lettuce，提供统一的 RedisTemplate API（也支持切换为 Jedis）。

## Spring Boot 集成

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>   <!-- Lettuce 连接池必需 -->
</dependency>
```

```yaml
spring:
  data:
    redis:              # Spring Boot 2.x 前缀是 spring.redis
      host: 192.168.1.100
      port: 6379
      password: 123321
      database: 0       # 默认 0
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
          max-wait: 1000ms
```

## RedisTemplate

Spring Boot 自动配置了 `RedisTemplate<Object, Object>` 和 `StringRedisTemplate`（key/value 都是 String）。五种类型各对应一个操作对象：

| API | 类型 |
| ---- | ---- |
| opsForValue() | String |
| opsForHash() | Hash |
| opsForList() | List |
| opsForSet() | Set |
| opsForZSet() | ZSet |
| delete() / expire() / keys() 等 | 通用命令 |

```java
@Autowired
private StringRedisTemplate redisTemplate;

// String：写入并设置 60s 过期
redisTemplate.opsForValue().set("code:13800001111", "9527", 60, TimeUnit.SECONDS);

// Hash：读写字段
redisTemplate.opsForHash().put("user:1", "name", "rose");
Object name = redisTemplate.opsForHash().get("user:1", "name");
```

## 序列化问题（重点坑）

`RedisTemplate<Object, Object>` 默认使用 **JdkSerializationRedisSerializer**：

- 存进去的是 JDK 序列化字节码，redis-cli 里**不可读**（乱码）。
- 体积大、占内存。
- 实体类必须实现 `Serializable`。

生产环境两种方案：

**方案一（推荐）：StringRedisTemplate + 手动 JSON 序列化**

```java
private static final ObjectMapper MAPPER = new ObjectMapper();

public void setUser(User user) throws JsonProcessingException {
    redisTemplate.opsForValue().set("user:1", MAPPER.writeValueAsString(user));
}

public User getUser() throws JsonProcessingException {
    String json = redisTemplate.opsForValue().get("user:1");
    return json == null ? null : MAPPER.readValue(json, User.class);
}
```

**方案二：自定义 RedisTemplate，值用 JSON 序列化器**

```java
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(factory);
    // key 用 String 序列化，value 用 JSON 序列化
    template.setKeySerializer(new StringRedisSerializer());
    template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
    template.setHashKeySerializer(new StringRedisSerializer());
    template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
    return template;
}
```

> `GenericJackson2JsonRedisSerializer` 会在 JSON 里额外记录 `@class` 类型信息以便自动反序列化；缺点是体积略增、且解析出的类型必须与当前类路径匹配。方案一更轻量可控，也是多数教程的选择。

结合 Spring Boot 详见 [[web后端开发/JavaWeb/springboot/springboot|springboot]]。
