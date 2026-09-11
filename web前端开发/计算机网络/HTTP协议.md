---
title: HTTP协议
source: https://www.yuque.com/mook-mvqcp/wc9wsu/ehy7qpf3o5k41qr4
created: 2026-05-28
updated: 2026-09-07
tags:
  - 计算机网络
---

> [!info] 目录说明
> 目录结构参照 MDN Web Docs 的 [HTTP 指南](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides) 与《图解 HTTP》，并补齐面试高频主题。标注「待补充」的小节为尚未填充内容，按需逐步完善。

## 一、HTTP 协议基础

### 1. 什么是 HTTP 协议

HTTP 协议是一种应用层协议，规定了在互联网上交换数据的格式和规则，用于传输诸如 HTML 文档之类的资源。

使用 HTTP 协议通信的双方，一方被称为服务端，一方被称为客户端，二者通过交换一个个独立的消息进行通信。客户端发起请求消息，服务端接收请求消息做出处理并返回响应消息。

客户端与服务端之间还存在一种叫代理的机器或者计算机参与 HTTP 消息的传递，这些代理可以发挥很多作用，比如缓存、过滤、负载均衡、认证等。

### 2. HTTP 请求方法

HTTP 请求方法表示对给定资源要执行的操作，HTTP 请求方法应该是安全的、幂等的、可缓存的。

除了以下 6 个之外，还有 3 个不常用的 HTTP 请求方法，分别是 CONNECT、TRACE 和 OPTIONS。

#### （1）GET

GET 方法请求一个指定资源的表示形式，用来获取数据。

#### （2）HEAD

HEAD 方法请求一个与 GET 请求响应相同的响应，但不包含响应体，即起始行、响应头相同但是没有响应体。

#### （3）PUT

PUT 方法用有效载荷请求替换目标资源的所有当前表示。

#### （4）POST

POST 方法用于将实体提交到指定的资源，通常导致在服务器上的状态变化或副作用。

#### （5）DELETE

DELETE 方法删除指定的资源。

#### （6）PATCH

PATCH 方法用于对资源应用部分修改。

**各方法的安全性、幂等性与可缓存性对比**（待补充）

### 3. HTTP 状态码（待补充）

五大类：1xx 信息、2xx 成功、3xx 重定向、4xx 客户端错误、5xx 服务端错误。

常见状态码：200、204、206、301、302、304、307、308、400、401、403、404、500、502、503、504。

**301 与 302 的区别、304 如何配合缓存工作**（待补充）

### 4. HTTP 报文的结构

HTTP 报文分为两类，一是请求报文，二是响应报文。

#### （1）请求报文

**起始行**：包含请求方法、请求目标和 HTTP 版本。

- 请求目标通常是一个 URL，或者是协议、域名和端口的绝对路径。
- HTTP 版本是期望的响应版本的指示符。

**标头（Header）**：不区分大小写的字符串，紧跟着一个冒号，其结构取决于标头的值，一个标头是一行。

- 可以有多个标头。
- 请求标头可以分为这几类：通用标头、请求标头、表示标头。
- 以一个空行指示标头区域的结束。

**主体（Body）**：主体是可选的，不是所有的请求都需要主体，这由使用什么请求方法以及携带什么数据决定。

主体可以分为两类：单一资源主体，由一个单文件组成，由两个标头定义（Content-Type 和 Content-Length）；多资源主体，由多部分主体组成，每一部分包含不同的信息。

#### （2）响应报文

**状态行**：响应报文的起始行被称为状态行。包含协议版本、状态码、状态文本。

**标头**：结构和请求标头一致，但是分类有点区别：通用标头、响应标头和表示标头。

**主体**：主体大致分为三类：单资源主体，由已知长度的单文件组成；单资源主体，由未知长度的单个文件组成，通过将 Transfer-Encoding 设置为 chunked 来使用分块编码；多资源主体，由多部分 body 组成，每部分包含不同的信息段。

### 5. HTTP 流（待补充）

一条 HTTP 连接上请求的完整过程：打开 TCP 连接 → 发送 HTTP 报文 → 服务器读取并处理 → 返回响应 → 关闭连接或复用。

### 6. HTTP 系统的构成（待补充）

客户端（浏览器等用户代理）、服务端，以及中间的代理：缓存代理、网关、负载均衡器。

### 7. HTTP 的特性（待补充）

简单、可扩展、无状态（Cookie 用于补充状态）、可靠（基于 TCP / QUIC）。

## 二、HTTP 报文的重要头部

### 1. 缓存

缓存是将请求得到的资源存储在本地上以备复用的手段，有利于减少网络资源消耗以及减轻服务器压力。

缓存分为强缓存和协商缓存。

#### （1）强缓存

强缓存是指缓存有效期内直接复用本地缓存，而不向服务器发送请求获取资源。

**关键标头：**

- Cache-Control（HTTP/1.1，优先级比 Expires 高）
- Expires（HTTP/1.0），指定一个绝对到期时间，一般作为 Cache-Control 的平替。

**Cache-Control 的常用指令**（待补充）：max-age、no-cache、no-store、private / public、must-revalidate。

#### （2）协商缓存

如果强缓存过期了，或者 Cache-Control 是 no-cache，浏览器向服务器发送请求询问服务器缓存的资源有没有发生更改，如果没有，继续复用本地的缓存；如果服务器发现资源已发生更改，那么返回新的资源，浏览器更新缓存。

有两对标头可以实现协商缓存，分别是 **ETag / If-None-Matched** 和 **Last-Modified / If-Modified-Since**。

ETag 代表资源的版本标识符，是一个响应标头。首次请求，服务器返回带有 ETag 的响应，浏览器存储这个 ETag；协商缓存阶段，浏览器发送带有 If-None-Matched 的请求，服务器重新计算资源的 ETag，对比 If-None-Matched，如果相等，返回 304（Not Modified）但是不返回资源主体；如果不相等，返回 200 OK、新的 ETag 与新的资源主体，浏览器更新缓存和 ETag。

Last-Modified / If-Modified-Since 原理差不多，但是是基于修改时间，准确性差一些。

### 2. 连接管理（待补充）

短连接、长连接（keep-alive）、HTTP/1.1 的队头阻塞问题与浏览器并发连接数限制。

### 3. CORS

因为同源策略的限制，浏览器（**服务器与服务器之间的互相访问没有这个限制**）只允许应用向同源（协议、域名和端口一致为同源）的站点发送请求，从而防止恶意网站通过脚本（可能是用户误点了恶意链接）访问隐私数据或者发送伪造请求等，保证了不同源的页面相互隔离，保护用户隐私。

CORS（Cross-Origin Resource Sharing），跨域资源共享，是基于 HTTP 头的机制，通过一组 HTTP 标头，允许 Web 应用服务器进行跨源访问控制，从而使跨源数据传输得以安全进行。

#### （1）功能概述

跨源资源共享标准定义了一组 HTTP 标头，允许服务器决定什么源站通过浏览器请求有权限访问资源。对于可能会对服务器的状态产生副作用的请求方法，浏览器必须先使用 OPTIONS 方法发起一个预检请求，从而获悉服务器是否允许访问该请求，确认服务器允许之后，才发起实际的 HTTP 请求。此外，服务器可以在预检的返回中通知浏览器是否需要携带身份凭证，例如 cookie、http 认证相关数据。

#### （2）访问控制场景的流程

Access-Control-Allow-Origin 的作用是告诉浏览器，允许哪个源（origin）的 JavaScript 代码读取本次响应的内容。

**简单请求**

浏览器发起的请求满足以下**条件**，则被视为简单请求：

a. 使用 GET、HEAD、POST 三个方法之一；

b. 除了被用户代理自动设置的标头字段（例如 Connection、User-Agent 或其他在 Fetch 规范中定义为禁用标头名称的标头），允许人为设置的字段为 Fetch 规范定义的对 CORS 安全的标头字段集合（Accept、Accept-Language、Content-Language、Content-Type 等）；

c. Content-Type 指定的媒体类型为以下三种之一：text/plain、multipart/form-data、application/x-www-form-urlencoded。

简单请求的**过程**为：

a. 浏览器自动为请求标头添加 Origin，值是请求所属的源。

b. 服务器检查 Origin，如果允许该源，则在响应标头添加 Access-Control-Allow-Origin，值为请求标头 Origin 的值或者是 *。

c. 浏览器获取响应，检查 Access-Control-Allow-Origin 是否匹配当前的源，如果匹配才允许当前的 JavaScript 读取响应，反之报错。

**预检请求**

如果请求不是简单请求，浏览器会先发送一个 OPTIONS 方法的预检请求（preflight），询问服务器是否允许实际请求。

预检请求的标头应包含（待补充）：Origin、Access-Control-Request-Method、Access-Control-Request-Headers。

预检响应的标头包含（待补充）：Access-Control-Allow-Origin、Access-Control-Allow-Methods、Access-Control-Allow-Headers、Access-Control-Max-Age。

如果预检通过，浏览器才发送实际请求。

**带凭证的请求**

在 Web 开发中，"带凭证的请求"指的是跨域请求时，浏览器自动附带用户在该目标域名下已存储的认证信息（如 Cookie、HTTP 认证头、客户端证书等）。默认情况下，跨域请求不携带任何凭证，这是浏览器的安全策略。若需要携带，必须客户端与服务端双方明确配合。

**携带凭证的过程：**

a. 客户端显式要求携带凭证（fetch 的 `credentials: 'include'`、XHR 的 `withCredentials`）。

b. 服务端放行并允许凭证，必须设置响应标头 `Access-Control-Allow-Origin: https://example.com`、`Access-Control-Allow-Credentials: true`。注意 Access-Control-Allow-Origin 不能是 *，必须精确指定请求的来源（即 `Origin` 请求头的值）。这是为了防止任意恶意网站都能利用用户凭证访问受保护资源。

如果请求不是简单请求（例如 PUT、DELETE 或自定义头），浏览器先发送 OPTIONS 预检。此时服务端也需要在预检响应中返回同样的 CORS 头。

### 4. Cookie（待补充）

Set-Cookie 与 Cookie 标头、属性（Expires/Max-Age、Domain、Path、Secure、HttpOnly、SameSite）。

→ 展开见 [[web前端开发/计算机网络/Cookie|Cookie]]，鉴权相关见 [[web前端开发/计算机网络/web鉴权方式|web 鉴权方式]]

### 5. Referer

`Referer` 请求头包含了当前请求页面的来源页面的地址，即表示当前页面是通过此来源页面里的链接进入的。服务端一般使用 `Referer` 请求头识别访问来源，可能会以此进行统计分析、日志记录以及缓存优化等。

### 6. 其他重要标头（待补充）

Content-Type / Accept 系列、Host、User-Agent、Content-Encoding / Accept-Encoding、Transfer-Encoding: chunked。

## 三、HTTP 协议版本

### 1. HTTP/1.0（待补充）

每请求一个 TCP 连接，短连接。

### 2. HTTP/1.1（待补充）

长连接（keep-alive）、管线化、Host 标头与虚拟主机、更多缓存控制、队头阻塞问题。

### 3. HTTP/2.0（待补充）

二进制分帧、多路复用、头部压缩 HPACK、服务器推送；TCP 层的队头阻塞。

### 4. HTTP/3.0（待补充）

基于 QUIC（UDP）：0-RTT 握手、连接迁移、彻底解决队头阻塞。

## 四、HTTPS

### 1. 加密方式（待补充）

对称加密与非对称加密、HTTPS 的混合加密方案。

### 2. 数字证书（待补充）

CA 与证书链、防中间人攻击的原理。

### 3. TLS 过程（待补充）

TLS 握手流程（TLS 1.2 RSA 握手与 ECDHE 握手、TLS 1.3 的 1-RTT）。

## 五、HTTP 常见面试问题

### 1. GET 和 POST 的区别（待补充）

### 2. HTTP 与 HTTPS 的区别（待补充）

### 3. 从输入 URL 到页面展示发生了什么（待补充）

DNS 解析 → TCP 连接 → TLS 握手 → 发送 HTTP 请求 → 服务器处理返回响应 → 浏览器解析渲染。

### 4. 什么是无状态协议，如何保存状态（待补充）

---

上级：[[web前端开发/计算机网络/计算机网络|计算机网络]]

## 笔记

- [[web前端开发/计算机网络/Cookie|Cookie]]
- [[web前端开发/计算机网络/web鉴权方式|web 鉴权方式]]
- [[web前端开发/计算机网络/Web漏洞|Web 漏洞]]
- [[web前端开发/计算机网络/网络请求API/网络请求API|网络请求 API]]
