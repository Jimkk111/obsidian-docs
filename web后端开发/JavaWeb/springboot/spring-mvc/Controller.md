---
title: Controller
source: https://www.yuque.com/mook-mvqcp/viz8gx/dlccz0rmxxtw41or
created: 2026-08-16
updated: 2026-09-10
tags:
  - JavaWeb
---

上级：[[web后端开发/JavaWeb/springboot/spring-mvc/spring-mvc|spring-mvc]]

表现层笔记：怎么声明接口、怎么接参数、怎么回响应。Controller 只做"接参 → 调 Service → 返回结果"，**业务逻辑写在 Service**。

## 一、@RestController

```java
@RestController                     // = @Controller + @ResponseBody
@RequestMapping("/users")
public class UserController { ... }
```

- `@Controller`：声明这是 MVC 控制器
- `@ResponseBody`：返回值不走视图解析，**直接写进响应体**（对象自动用 Jackson 序列化成 JSON）

前后端分离项目一律用 `@RestController`。只写 `@Controller` 的话，返回值会被当成"视图名"去渲染页面（传统 JSP 套路）。

## 二、接口声明：@RequestMapping 系

| 注解 | 等价于 | 语义 |
|---|---|---|
| `@GetMapping` | `@RequestMapping(method = GET)` | 查询 |
| `@PostMapping` | `@RequestMapping(method = POST)` | 新增 |
| `@PutMapping` | `@RequestMapping(method = PUT)` | 修改 |
| `@DeleteMapping` | `@RequestMapping(method = DELETE)` | 删除 |

RESTful 风格：**URL 表示资源，HTTP 方法表示动作**。

```
GET    /users        查列表
GET    /users/1      查单个
POST   /users        新增（JSON 在请求体）
PUT    /users/1      修改
DELETE /users/1      删除
```

## 三、接收参数

### 1. 简单参数：参数名对上就自动绑定

```
GET /users?name=张三&age=18
```

```java
@GetMapping
public List<User> list(String name, Integer age) { ... }
// ?name=xx&age=18 → 同名自动类型转换并绑定
```

名字对不上 / 要设默认值 / 要可选，用 `@RequestParam`：

```java
@GetMapping
public List<User> list(@RequestParam("userName") String name,        // 请求参数名叫 userName
                       @RequestParam(defaultValue = "1") Integer page,
                       @RequestParam(required = false) Integer size) { ... }
```

### 2. 实体对象参数：参数多时打包

```java
@Data
public class UserQuery {
    private String name;
    private Integer age;
}

@GetMapping
public List<User> list(UserQuery query) { ... }
// ?name=张三&age=18 → 按 setter 逐个装进对象（不是 JSON 场景）
```

### 3. @PathVariable：路径里的变量

```java
@GetMapping("/{id}")
public User getById(@PathVariable Long id) { ... }
// GET /users/1 → id = 1
```

### 4. @RequestBody：接收 JSON 请求体

```java
@PostMapping
public void add(@RequestBody User user) { ... }
/*
POST /users
Content-Type: application/json

{"name": "张三", "age": 18}
*/
```

⚠️ 三个要点：

- `@RequestBody` 一个方法**只能标一个参数**（请求体只能读一次）
- 客户端必须带 `Content-Type: application/json`，否则 415
- JSON 字段名要和实体属性名对上（可用 `@JsonProperty` 映射）

### 5. 日期参数

```java
// URL 上的日期：
@GetMapping
public List<User> list(@DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate begin) { ... }

// JSON 里的日期：字段上加
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
private LocalDateTime createTime;
```

### 6. 其他（了解）

`@RequestHeader("Authorization")` 拿请求头、`@CookieValue` 拿 Cookie。

## 四、返回统一响应格式

约定一个统一结构，前端处理逻辑只写一遍：

```java
@Data
public class Result<T> {
    private Integer code;      // 0 或 200 成功，其他失败
    private String message;
    private T data;

    public static <T> Result<T> success(T data) {
        Result<T> r = new Result<>();
        r.code = 0; r.message = "success"; r.data = data;
        return r;
    }

    public static <T> Result<T> error(Integer code, String message) {
        Result<T> r = new Result<>();
        r.code = code; r.message = message;
        return r;
    }
}
```

```java
@GetMapping("/{id}")
public Result<User> getById(@PathVariable Long id) {
    return Result.success(userService.getById(id));
}
```

失败时不用每个接口都 try-catch，配合全局异常处理：[[web后端开发/JavaWeb/springboot/spring-mvc/异常处理|异常处理]]。

## ⚠️ 常见坑

| 现象 | 原因 |
|---|---|
| 400 Bad Request | 参数名不匹配、类型转换失败（`age=abc`）、缺必填参数 |
| 415 Unsupported Media Type | 发 JSON 但没带 `Content-Type: application/json` |
| 收到的字段全是 null | JSON 字段名和属性名不一致；或没有 setter |
| 日期收不到/格式错 | 没加 `@DateTimeFormat`（URL）或 `@JsonFormat`（JSON） |
| 接口 404 | 类上/方法上的路径拼错；`context-path` 前缀没算 |
