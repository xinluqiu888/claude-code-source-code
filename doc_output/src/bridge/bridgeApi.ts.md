# bridgeApi.ts — Bridge HTTP API 客户端封装

> **一句话总结**：封装与 Anthropic CCR 后端通信的所有 HTTP API 调用，包括环境注册、工作轮询、心跳、会话归档等操作，并统一处理 OAuth 认证、401 重试和错误分类。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/bridge/bridgeApi.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 540 |
| 主要职责 | 创建并返回 `BridgeApiClient` 实例，封装所有 bridge REST API 调用；处理认证刷新与错误状态转换 |

---

## 功能概述

`bridgeApi.ts` 是 bridge 模块的 HTTP 通信核心。它通过 `createBridgeApiClient` 工厂函数构建一个 `BridgeApiClient` 对象，将所有与 Anthropic Remote Control（CCR）后端的 REST API 调用集中在一处。

该文件的核心设计目标是"有且仅有一处 API 调用逻辑"。之前这些调用分散在各处，现在统一由此文件管理。所有调用都会自动附带认证头（OAuth Bearer Token 或环境密钥），并在收到 HTTP 401 时尝试一次 OAuth 刷新后重试。

错误处理上，将 HTTP 错误转换为两类：`BridgeFatalError`（不可恢复，如 401/403/404/410）和普通 `Error`（可重试，如 429 或 5xx）。调用方可据此决定是退出还是重试。

---

## 核心内容详解

### 导入与依赖

| 依赖 | 用途 |
|------|------|
| `axios` | 发起 HTTP 请求 |
| `./debugUtils.js` | `debugBody`（日志格式化）、`extractErrorDetail`（提取服务端错误信息） |
| `./types.js` | `BridgeApiClient`、`BridgeConfig`、`PermissionResponseEvent`、`WorkResponse` 等类型 |

### 主要类/函数/接口

#### `BridgeFatalError`
继承自 `Error` 的自定义错误类，携带 `status`（HTTP 状态码）和 `errorType`（服务端错误类型字符串，如 `"environment_expired"`）。用于标识不应重试的致命错误。

#### `validateBridgeId(id, label)`
对服务端返回的 ID（用于 URL 路径拼接）做安全校验，只允许 `[a-zA-Z0-9_-]+` 字符，防止路径穿越攻击。

#### `createBridgeApiClient(deps)`
工厂函数，返回 `BridgeApiClient` 对象，包含以下方法：

| 方法 | HTTP 动作 | 说明 |
|------|-----------|------|
| `registerBridgeEnvironment` | POST `/v1/environments/bridge` | 注册（或复用）一个 bridge 环境，返回 `environment_id` 和 `environment_secret` |
| `pollForWork` | GET `/v1/environments/{id}/work/poll` | 长轮询拉取待处理工作，返回 `WorkResponse` 或 `null` |
| `acknowledgeWork` | POST `/v1/environments/{id}/work/{wid}/ack` | 确认已接收工作，防止服务端重发 |
| `stopWork` | POST `/v1/environments/{id}/work/{wid}/stop` | 停止工作（可强制） |
| `deregisterEnvironment` | DELETE `/v1/environments/bridge/{id}` | 优雅关闭时注销环境 |
| `archiveSession` | POST `/v1/sessions/{id}/archive` | 归档会话，409 表示已归档（幂等） |
| `reconnectSession` | POST `/v1/environments/{id}/bridge/reconnect` | `--session-id` 恢复时重连会话 |
| `heartbeatWork` | POST `/v1/environments/{id}/work/{wid}/heartbeat` | 发送心跳延续租约 |
| `sendPermissionResponseEvent` | POST `/v1/sessions/{id}/events` | 发送权限响应（control_response）事件 |

#### `withOAuthRetry<T>(fn, context)`
内部辅助函数。执行一次请求，若得到 401 则调用 `deps.onAuth401` 刷新 Token 后再重试一次（与 `withRetry.ts` 相同模式）。

#### `handleErrorStatus(status, data, context)`
将非 200/204 的 HTTP 状态码转换为具体错误：
- 401 → `BridgeFatalError`（附登录提示）
- 403 → `BridgeFatalError`（过期则特殊提示）
- 404/410 → `BridgeFatalError`（会话/环境过期）
- 429 → 普通 `Error`（速率限制）
- 其他 → 普通 `Error`

#### `isExpiredErrorType(errorType)`
判断错误类型字符串是否表示会话/环境过期（包含 `"expired"` 或 `"lifetime"`）。

#### `isSuppressible403(err)`
判断 `BridgeFatalError` 是否为可静默的 403 错误（如 `external_poll_sessions` 或 `environments:manage` 权限不足，不影响核心功能）。

### 数据流与逻辑流程

```
调用方（bridgeMain / replBridge）
    ↓
createBridgeApiClient(deps) → BridgeApiClient
    ↓
每次 API 调用：
  1. resolveAuth() → 从 deps.getAccessToken() 获取 Bearer Token
  2. getHeaders() → 构建认证/版本头（含 X-Trusted-Device-Token 如有）
  3. withOAuthRetry() / 直接 axios 调用
  4. handleErrorStatus() → 成功放行，错误抛出
  5. debug() 记录请求/响应摘要（敏感字段自动脱敏）
```

### 对外接口（导出内容）

| 导出 | 类型 | 说明 |
|------|------|------|
| `validateBridgeId` | 函数 | 校验 URL 路径 ID 安全性 |
| `BridgeFatalError` | 类 | 不可重试的致命 bridge 错误 |
| `createBridgeApiClient` | 函数 | 创建 API 客户端实例 |
| `isExpiredErrorType` | 函数 | 判断错误是否为过期类型 |
| `isSuppressible403` | 函数 | 判断 403 是否可静默 |

---

## 设计要点

- **依赖注入**：所有外部依赖（Token 获取、401 处理、受信设备 Token）通过 `BridgeApiDeps` 注入，便于测试和隔离。
- **错误分层**：`BridgeFatalError` vs 普通 `Error` 的双层错误体系，让调用方无需解析 HTTP 状态码即可决策重试策略。
- **空轮询抑制**：`consecutiveEmptyPolls` 计数器控制日志频率，避免每次空轮询都打印日志（每 100 次才输出一次）。
- **安全 ID 校验**：服务端返回的 ID 在拼入 URL 前强制通过 `validateBridgeId` 白名单校验，防止注入。

---

## 与其他文件的关系

- **依赖**：
  - `./debugUtils.js`：日志格式化与错误提取
  - `./types.js`：所有类型定义
  - `axios`：HTTP 客户端
- **被依赖**：
  - `bridgeMain.ts`：主 bridge 循环使用此客户端
  - `replBridge.ts`：REPL bridge 核心使用此客户端
  - `bridgeDebug.ts`：故障注入时包装此客户端

---

## 注意事项

- `pollForWork` 使用 `environmentSecret` 而非 OAuth Token（因为轮询频率很高，使用环境密钥可绕过 DB 查用户）。心跳也同样使用 JWT/环境密钥。
- Daemon 调用方（使用环境变量 Token）不传 `onAuth401`，因此 Token 不能刷新，401 直接抛 `BridgeFatalError`。
- 409 归档状态被视为幂等成功，不抛出错误。
- `BETA_HEADER = 'environments-2025-11-01'` 是必须的 API Beta 版本标识。
