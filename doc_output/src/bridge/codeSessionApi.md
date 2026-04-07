# codeSessionApi.ts — CCR v2 代码会话 API 的 HTTP 包装器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/bridge/codeSessionApi.ts`
- **文件类型**: TypeScript 模块
- **行数**: 约 169 行
- **主要职责**: 提供 CCR v2 代码会话 API 的 HTTP 包装器，包括会话创建和远程凭证获取

## 功能概述

`codeSessionApi.ts` 提供了 Claude Code Remote (CCR) v2 代码会话 API 的轻量级 HTTP 包装器。该模块与 `remoteBridgeCore.ts` 分离，以便 SDK 的 `/bridge` 子路径可以导出 `createCodeSession` 和 `fetchRemoteCredentials` 功能，而无需打包沉重的 CLI 树（分析、传输等）。

调用者需要提供显式的访问令牌和基础 URL——模块不进行隐式认证或配置读取。这种设计使得模块可以被 SDK 和 CLI 共同使用，同时保持依赖最小化。

模块包含两个主要功能：创建代码会话（`createCodeSession`）和获取远程凭证（`fetchRemoteCredentials`）。前者用于在桥接环境中创建新会话，后者用于获取 WebSocket 连接所需的 JWT 凭证。

## 核心内容详解

### 常量定义

| 常量 | 值 | 说明 |
|------|-----|------|
| `ANTHROPIC_VERSION` | `'2023-06-01'` | API 版本头 |

### oauthHeaders 辅助函数

**签名**: `oauthHeaders(accessToken: string): Record<string, string>`

构建 OAuth 请求头：
- `Authorization`: `Bearer {accessToken}`
- `Content-Type`: `application/json`
- `anthropic-version`: `2023-06-01`

### createCodeSession 函数

**签名**: 
```typescript
createCodeSession(
  baseUrl: string,
  accessToken: string,
  title: string,
  timeoutMs: number,
  tags?: string[]
): Promise<string | null>
```

**功能**: 通过 POST `/v1/code/sessions` 创建代码会话。

**请求体**:
```typescript
{
  title: string,
  bridge: {},  // 桥接模式的正向信号
  ...(tags?.length ? { tags } : {})
}
```

**响应处理**:
- 预期状态码: 200 或 201
- 响应体需包含 `session.id` 字段
- ID 格式要求: 以 `cse_` 开头
- 失败返回 `null`（非致命错误）

**错误处理**:
- 网络错误/超时: 记录调试日志，返回 null
- HTTP 4xx/5xx: 提取错误详情，记录调试日志，返回 null
- 响应格式错误: 记录调试日志（截断到 200 字符），返回 null

### fetchRemoteCredentials 函数

**签名**:
```typescript
fetchRemoteCredentials(
  sessionId: string,
  baseUrl: string,
  accessToken: string,
  timeoutMs: number,
  trustedDeviceToken?: string
): Promise<RemoteCredentials | null>
```

**功能**: 通过 POST `/v1/code/sessions/{id}/bridge` 获取远程凭证。

**RemoteCredentials 类型**:
```typescript
{
  worker_jwt: string;      // JWT 令牌（不透明，不解码）
  api_base_url: string;    // WebSocket API 基础 URL
  expires_in: number;      // JWT 有效期（秒）
  worker_epoch: number;    // 工作器周期（服务器端递增）
}
```

**请求头**: 
- 基础 OAuth 头
- `X-Trusted-Device-Token`: 可选，受信任设备令牌

**worker_epoch 处理**:
ProtoJSON 将 `int64` 序列化为字符串以避免 JS 精度损失，但 Go 可能返回数字。实现处理两种格式：
```typescript
const rawEpoch = data.worker_epoch
const epoch = typeof rawEpoch === 'string' ? Number(rawEpoch) : rawEpoch
```

验证 `epoch` 是有限安全整数。

**响应验证**:
检查所有必需字段存在且类型正确：`worker_jwt`（字符串）、`expires_in`（数字）、`api_base_url`（字符串）、`worker_epoch`（数字或字符串）。

## 设计要点

1. **显式依赖**: 所有认证和配置通过参数传递，不读取全局状态或环境变量。这使得模块可被 SDK 复用。

2. **非致命错误**: 所有失败返回 `null` 而非抛出，调用者决定如何处理失败（重试、降级或报错）。

3. **详细调试**: 错误信息记录到调试日志，包括 HTTP 状态码、错误详情和响应预览（截断到 200 字符）。

4. **JWT 不透明**: `worker_jwt` 被视为不透明令牌，模块不解码或验证 JWT 内容。

5. **类型安全**: 使用运行时验证确保响应格式符合预期，避免后续代码因类型错误崩溃。

6. **ProtoJSON 兼容**: 处理 `int64` 的字符串/数字双格式，兼容不同后端编码。

7. **受信任设备支持**: 可选的 `trustedDeviceToken` 参数支持受信任设备流。

8. **Axios 配置**: 使用 `validateStatus: s => s < 500` 将 5xx 错误视为可重试，4xx 视为客户端错误。

## 与其他文件的关系

| 文件 | 关系描述 |
|------|----------|
| `./debugUtils.js` | 导入 `extractErrorDetail` 提取 API 错误详情 |
| `../utils/debug.js` | 使用 `logForDebugging` 记录调试信息 |
| `../utils/errors.js` | 导入 `errorMessage` 提取错误消息 |
| `../utils/slowOperations.js` | 使用 `jsonStringify` 序列化响应用于调试 |

## 注意事项

1. **baseUrl 格式**: `baseUrl` 应包含协议和域名，不包含路径后缀（如 `https://api.anthropic.com`）。

2. **cse_ 前缀**: `createCodeSession` 要求响应 ID 以 `cse_` 开头，这是 v2 API 的会话 ID 格式。v1 可能使用不同前缀。

3. **bridge 字段**: 请求体中的 `bridge: {}` 是正信号，省略或发送空字符串会导致 400 错误。

4. **超时行为**: `timeoutMs` 传递给 axios，但包含连接和读取超时。实际超时可能略长于指定值。

5. **worker_epoch 语义**: 每次 `/bridge` 调用服务器端递增，可用作工作器注册表和会话一致性检查。

6. **受信任设备令牌**: 如果提供，应是与用户账户关联的有效令牌。服务器验证令牌有效性。

7. **SDK 使用**: 作为 SDK 导出的一部分时，调用者需要自行管理访问令牌生命周期（刷新、过期处理）。

8. **错误详情提取**: 使用 `extractErrorDetail` 检查 `data.message` 或 `data.error.message`，不同端点可能使用不同格式。
