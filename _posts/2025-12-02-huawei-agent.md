---
title: 华为小艺接入分析
author: 一三
date: 2025-12-03 017:00:00 +0800
categories: [Work]
tags: [analysis]
---

# 小艺 A2A 模式开发规范（速览）
小艺是2025年11月份开放的A2A能力，允许用户通过小艺访问第三方开发的agent服务。（目前应该还是需要和商务联系，这个没有广泛开放）

[华为官方文档](https://developer.huawei.com/consumer/cn/doc/service/a2a-basic-configuration-0000002437785686)
需要在创建好账号之后，在小艺开发平台通过A2A模式创建智能体，配置智能体的名称，智能体描述（用于做意图分发），API URL，会话维持的方式，以及认证方式等


面向本地 Agent Server 适配华为小艺 A2A 协议的开发说明，便于后端快速对接和自测。

## 1. 接口形态
- 单一入口：`POST /agent/message`
- 协议：JSON-RPC 2.0
- 必要 Header：
  - `Content-Type: application/json`
  - `agent-session-id`（除 `initialize` 外必需）
  - `Authorization`（仅 `initialize` 用，校验规则自定）
- 返回：SSE（`text/event-stream`），每帧是 JSON-RPC 成功响应，`result` 为状态或内容事件。

## 2. 支持的 method
- `initialize`：生成并返回 `agentSessionId`（可映射到业务侧 session，如 `chatSession/create`）。
- `notifications/initialized`：初始化完成回调，HTTP 200。
- `message/stream`：核心流式接口。
- `tasks/cancel`：取消当前任务。
- `clearContext`：清理会话/上下文（可调用业务侧删除接口，如 `chatSession/delete`，按需实现）。
- `authorize` / `deauthorize`：账号授权绑定/解绑（示例占位，根据业务对接）。

## 3. message/stream 请求格式
```jsonc
{
  "jsonrpc": "2.0",
  "id": "msg-1",            // 本次 RPC 唯一 ID
  "method": "message/stream",
  "params": {
    "id": "task-001",       // 任务 ID（整场流式交互内保持不变）
    "sessionId": "sess-1",  // 客户端分配的会话 ID，切换即视为新上下文（服务端只负责管理，不负责生成）
    "agentLoginSessionId": "login-xxx", // 用户登录态（若有）
    "message": {
      "role": "user",
      "parts": [
        { "kind": "text", "text": "用户输入的 Query" }
        // { "kind": "file", "file": { name, mimeType, bytes|uri } }
        // { "kind": "data", "data": { ...结构化数据... } }
      ]
    }
  }
}
```

## 4. 服务端返回事件类型（SSE 帧中的 result）
- `TaskStatusUpdateEvent`（状态事件）
  - 字段：`taskId`、`kind=status-update`、`final`、`status.state`（submitted|working|input-required|completed|canceled|failed|unknown）、`status.message.parts[]`（可填进度描述）。
- `TaskArtifactUpdateEvent`（内容事件）
  - 字段：`taskId`、`kind=artifact-update`、`append`、`lastChunk`、`final`、`artifact.artifactId`、`artifact.parts[]`。

### parts 类型
- `text`：正文，支持 markdown。
- `reasoningText`：思考过程。
- `data`：结构化数据（卡片、跳转等，见下）。
- `file`：文件（与场景相关时使用）。

## 5. 流式语义建议
- 首帧：返回 `status-update state=working final=false`。
- 中间帧：多次 `artifact-update append=true lastChunk=false final=false`。
- 结束帧：补一个 `status-update state=completed final=true`（若有内容，最后一条 `artifact-update` 的 `lastChunk=true final=false` 也可配合）。
- 取消：收到 `tasks/cancel` 后返回 `status-update state=canceled final=true`，并停止继续推送。
- 失败：返回 `status-update state=failed final=true`，`message` 中带错误原因。

## 6. 跳转/卡片（kind=data）
在 `artifact.parts` 里增加：
```jsonc
{
  "kind": "data",
  "data": {
    // commands：触发端能力
    "commands": [
      { "header": { "namespace": "demo", "name": "openLink" }, "payload": { "url": "https://..." } }
    ],
    // cardsInfo：业务卡片，卡片模板需在小艺开放平台注册
    "cardsInfo": { "cardName": "service_link", "cardData": { "items": [{ "title": "查看详情", "url": "https://..." }] }, "displayType": "DisplayFaCard" },
    // chipsInfo：底部 Chips，可用 superlink/h5/app 小程序协议
    "chipsInfo": { "displayChips": { "chipsList": [{ "content": "superlink://vassistant?text=打开H5", "domain": "agent" }] } },
    // reference：引用/链接卡片，webLink.url 可放 H5/小程序地址，startMode 区分跳转方式（需与客户端约定）
    "reference": {
      "items": [{
        "params": { "name": "h5_link", "source": "agent" },
        "card": {
          "type": "webLink",
          "params": {
            "title": "前往H5",
            "link": { "webLink": { "url": "https://...", "startMode": 1 } }
          }
        }
      }]
    }
  }
}
```
> 提示：`displayType`、`startMode` 等需按客户端约定取值；卡片模板名需在小艺侧已注册。

## 7. 清理会话 clearContext
- 建议映射到业务侧会话删除（如 `POST /api/ai/chat/chatSession/delete`），携带 `chatSessionId/agentSessionId`。若接口未验证，请在日志中标记。
- 服务端可保持无状态，客户端通过更换 `sessionId` 视为新上下文。

## 8. 最小自测清单
- `initialize` 正常返回 `agentSessionId`。
- `message/stream` 能产出：working 状态 -> 若干 artifact-update -> completed 状态。
- 取消：`tasks/cancel` 后收到 canceled 状态，后续不再推送。
- 失败场景：远端报错时返回 failed 状态。
- `clearContext` 能正常透传/调用业务删除接口（如有）。
- 返回的 `data` 卡片字段符合客户端约定（命名、模板、startMode）。 


## 9. 常见问题
> ### sessionID,agentSessionId,agentLoginSessionId，分别是怎么管理的？
sessionID: 是由客户端（小艺）在发起message/stream的时候带上的，用于记录真实的会话请求，服务端没有创建权限（可能华为内部后期会允许同一个sessionID使用不同的智能体来实现完整的功能）

agentSessionId： 客户端打开第三方智能体界面的时候，会尝试尝试进行初始化，获取对应的agentSessionId（原文，未确认），从个人理解方面，这个应该是用来标记当前客户端启动的针对于第三方智能体的SessionID，并非实际会话的SessionId

agentLoginSessionId： 原文说明为智能体内部用于管理用户的一次登录的，个人认为智能体内部用于管理用户登录使用的，比方说，数据库键使用用户sessionid，内部保存UserInfo,accessToken等信息。（华为要求第三方agent提供deauthorize功能，以确保因此安全）
