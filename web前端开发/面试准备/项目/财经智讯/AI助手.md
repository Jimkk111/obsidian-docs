---
title: AI助手
source: https://www.yuque.com/mook-mvqcp/wc9wsu/xwcy9hvvi9y2hug1
created: 2026-05-27
updated: 2026-05-27
tags:
  - 面试准备
---

## STAR 法则：AI 智能助手（Chatbot）集成

### S — 情境（Situation）

在一个财经新闻类 Vue 3 前端项目中，需要集成 AI 对话能力，让用户可以通过自然语言获取财经资讯解读、投资知识问答等服务。痛点在于：后端大模型推理耗时较长（数秒到数十秒），传统的一次性返回会导致用户长时间面对空白页面，交互体验差。

### T — 任务（Task）

我负责从零搭建完整的 AI 聊天模块，核心目标是：

1. 基于 SSE 实现流式输出，让 AI 回复像 ChatGPT 一样逐字呈现

2. 基于 Pinia 设计消息状态追踪和会话管理机制

3. 实现 localStorage 持久化，确保用户刷新后能恢复会话

### A — 行动（Action）

**流式输出（SSE）方面**，我选用了原生 `fetch` API 而非 Axios。原因是 Axios 基于 XHR，不支持 ReadableStream 的流式消费。我封装了 `createStreamingChat` 函数（src/api/ai.ts），通过 `response.body.getReader()` 逐块读取 SSE 数据，按 `data:` 前缀解析每一行，遇到 `[DONE]` 标记时结束流。同时引入 `AbortController` 返回取消函数，为后续的"停止生成"功能预留了接口。中间层 `chatCompletion`（src/services/aiService.ts）将流式过程包装成 Promise，使得调用方可以通过 `onChunk` 回调逐段获取内容，同时保持 async/await 的使用体验。

**状态管理方面**，我使用 Pinia Composition API 风格的 Store（src/stores/aiSession.ts）集中管理：会话列表、当前会话、消息数组、加载/发送状态、错误信息和侧边栏开关。消息对象有 `status` 字段（`streaming | complete | error`），UI 根据状态渲染不同效果——流式时显示打字机光标和逐字动画，完成后显示复制/重新生成按钮。会话创建采用**惰性策略**：点击"新建对话"时只清空本地状态，真正发送第一条消息时才调用后端创建会话，避免了空会话污染列表。

**持久化方面**，我将 `lastSessionId` 和会话元数据缓存到 localStorage。初始化时优先从服务端加载会话列表，网络异常时自动降级到本地缓存，保证了离线场景下的基本可用性。

**UI 交互方面**，实现了会话侧边栏（按今天/昨天/本周分组、搜索过滤、内联重命名、滑动删除）、消息的 Markdown 渲染（基于 `marked` 库）、流式打字机效果（30ms/字符的 setInterval）、以及"重新生成"功能（回退最后一轮对话后重新发送）。

### R — 结果（Result）

最终实现了一个功能完整的 AI 助手模块，包含 7 个 Vue 组件和完整的 Store-Service-API 三层架构。流式输出将首次响应时间（TTFB）后的体感等待从数十秒缩短到即时可见，用户能实时看到 AI 的生成进度。会话管理和持久化机制使用户可以无缝切换多轮对话、刷新不丢失上下文，整体交互体验对标主流 AI 聊天产品。

---

**面试小贴士**：如果面试官追问，可以延伸讲 **AbortController 取消机制的设计考量**（为什么预留了但没接 UI——因为流式响应中前端 abort 后后端可能仍在推理，需要后端配合支持真正的取消）、**为什么用 fetch 而不是 EventSource**（EventSource 只支持 GET 且无法自定义请求头，而聊天需要 POST 发送消息体并携带 JWT token），以及**消息去重和乐观更新的思路**。
