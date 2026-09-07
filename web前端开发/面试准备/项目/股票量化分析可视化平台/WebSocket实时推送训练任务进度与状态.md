---
title: WebSocket实时推送训练任务进度与状态
source: https://www.yuque.com/mook-mvqcp/wc9wsu/clswawd3866wlmsl
created: 2026-05-27
updated: 2026-05-27
tags:
  - 面试准备
---

# 一、面试回答

**Situation：** 训练 LSTM 模型是异步长时间任务，可能跑几十到几百轮 epoch、耗时几分钟到几十分钟。如果只靠一次 HTTP 请求轮询结果，用户看不到中间过程，点击"开始训练"后只能干等，体验极差。同时服务端需要感知客户端是否中途离开，避免无效计算浪费 GPU 资源。

**Task：** 实现训练进度的实时推送与展示——让用户能看到每一轮 epoch 的 loss 变化趋势、当前进度百分比、最佳指标等，并支持随时取消正在进行的训练。

**Action：**

- **协议选型与连接建立。** 选用 HTTP + WebSocket 分工协作的方案。前端通过 POST 请求提交训练参数（数据源、网络层配置、超参），服务端创建异步训练任务后立即返回一个唯一 `jobId`，不阻塞等待训练完成。前端拿到 `jobId` 后立即 `new WebSocket` 连接到 `/api/training/{jobId}/ws`，浏览器底层自动完成 HTTP Upgrade 握手——携带 `Connection: Upgrade` 和 `Upgrade: websocket` 头，服务端将 `Sec-WebSocket-Key` 与固定魔术字符串拼接做 SHA-1 编码后返回，校验通过后协议切换为 WebSocket，状态码 101。

- **应用层消息协议设计。** 服务端推送 JSON 文本帧，每条消息带 `type` 字段区分阶段：`started`（训练启动，带总 epoch 数）、`epoch`（每轮完成，带当前 loss、val_loss、最佳 loss 和 epoch）、`completed`（训练正常结束，带最终指标和耗时）、`failed`（训练异常终止，带错误原因）。前端 `onmessage` 事件做 `switch-case` 分发：`started` 初始化进度总轮次；`epoch` 是核心通路，每收到一条就将 loss 数据追加到日志数组、更新当前进度和最佳指标，曲线图和进度条随之重绘；`completed` 关闭 WebSocket 并走 HTTP GET 拉取完整训练详情写入结果区；`failed` 关闭连接并展示错误。

- **连接保活。** 协议层 Ping/Pong 帧由浏览器自动回复，无需代码干预。但 Nginx 等反向代理对 WebSocket 的空闲超时可能不感知协议帧，所以应用层额外约定 `ping` 消息——服务端定期推送，前端收到后直接 `break` 不做渲染，纯粹为了让链路上有数据流过，防止被代理断开。

- **取消训练。** 前端取消按钮直接调用 `ws.close()`，浏览器发送关闭帧（opcode `0x8`）。服务端 WebSocket 的 `onclose` 事件触发后，根据 `jobId` 找到正在运行的后台训练进程并终止，释放 GPU 资源。全程不需要额外设计一个取消 API——WebSocket 连接本身的生命周期就是任务的生命周期。

**Result：** 用户点击"开始训练"后能实时看到 loss 曲线逐轮下降、进度条平滑推进，训练过程完全透明。中途取消只需点一次按钮，服务端即时释放资源。整套方案复用了 WebSocket 协议内建的握手、心跳和关闭机制，代码层面只有几十行，但功能做到实时、可控、可靠。

**Situation：** 训练任务是异步长时间运行的（几十到几百轮 epoch），不可能通过一次 HTTP 请求等结果，用户需要在训练过程中看到实时进度。

**Task：** 实现训练进度的实时推送与展示，并支持用户中途取消训练。

**Action：** 采用"HTTP 建任务 + WebSocket 推进度"的方案。前端 POST 请求创建训练任务拿到 ID 后，立即建立 WebSocket 长连接。服务端按训练阶段推送不同 type 的 JSON 消息——`started` 初始化总轮次，`epoch` 逐轮携带 loss 和 val_loss，`completed` 或 `failed` 标记结束。前端 `onmessage` 按 type 分发处理，驱动损失曲线、进度条和关键指标实时更新。WebSocket 协议层的 Ping/Pong 由浏览器自动处理，应用层额外用 ping 消息防反向代理断开。取消训练通过关闭 WebSocket 连接实现，服务端检测到断开后终止后台任务。

**Result：** 用户能实时看到每一轮 epoch 的 loss 变化趋势，训练过程透明可控，中途取消无需额外 API，一关连接即可。
