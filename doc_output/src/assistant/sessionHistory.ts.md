# sessionHistory.ts — 会话历史管理

> **一句话总结**：提供会话事件历史的分页获取功能，支持获取最新事件和更早事件。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/assistant/sessionHistory.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 88 |
| 主要职责 | 封装会话历史事件的分页获取API调用 |

---

## 功能概述

该文件实现了与会话历史相关的数据获取功能，主要用于从远程API获取特定会话的事件历史记录。它提供了一种分页机制，允许客户端按需获取最新事件或更早的历史事件。

该模块是Claude Code中会话管理功能的一部分，支持通过OAuth认证与后端API通信，获取结构化的会话消息数据。

---

## 核心内容详解

### 导入与依赖

| 依赖 | 来源 | 用途 |
|------|------|------|
| `axios` | 外部库 | HTTP请求客户端 |
| `getOauthConfig` | `../constants/oauth.js` | 获取OAuth配置信息 |
| `SDKMessage` | `../entrypoints/agentSdkTypes.js` | SDK消息类型定义 |
| `logForDebugging` | `../utils/debug.js` | 调试日志记录 |
| `getOAuthHeaders`, `prepareApiRequest` | `../utils/teleport/api.js` | OAuth认证相关工具 |

### 常量定义

- **`HISTORY_PAGE_SIZE`** (number): 默认每页事件数量，固定值为 **100**

### 类型定义

#### `HistoryPage`
会话历史页面对象
- `events`: `SDKMessage[]` - 按时间顺序排列的事件数组
- `firstId`: `string | null` - 当前页最早事件的ID（用于分页游标）
- `hasMore`: `boolean` - 是否存在更早的事件

#### `SessionEventsResponse`
API响应结构（内部使用）
- `data`: `SDKMessage[]` - 事件数据数组
- `has_more`: `boolean` - 是否有更多数据
- `first_id`/`last_id`: 分页游标ID

#### `HistoryAuthCtx`
认证上下文对象
- `baseUrl`: API基础URL
- `headers`: 请求头配置

### 主要函数

#### `createHistoryAuthCtx`
- **类型**: `async function`
- **用途**: 初始化认证上下文，一次性准备认证信息和请求头
- **参数**: 
  - `sessionId`: `string` - 目标会话ID
- **返回值**: `Promise<HistoryAuthCtx>` - 认证上下文
- **关键逻辑**: 
  - 调用 `prepareApiRequest()` 获取访问令牌和组织UUID
  - 构建API基础URL：`/v1/sessions/{sessionId}/events`
  - 设置请求头，包含OAuth令牌和组织信息
  - 添加Beta特性标识头 `anthropic-beta: ccr-byoc-2025-07-29`

#### `fetchPage`
- **类型**: `async function` (内部使用)
- **用途**: 执行分页数据的HTTP GET请求
- **参数**: 
  - `ctx`: `HistoryAuthCtx` - 认证上下文
  - `params`: 查询参数对象
  - `label`: 调试标签
- **返回值**: `Promise<HistoryPage | null>`
- **关键逻辑**:
  - 使用axios发送GET请求，15秒超时
  - 接受任何HTTP状态码（`validateStatus: () => true`）
  - 非200响应记录调试日志并返回null
  - 成功响应转换为 `HistoryPage` 格式

#### `fetchLatestEvents`
- **类型**: `async function`
- **用途**: 获取最新的会话事件（按时间倒序）
- **参数**: 
  - `ctx`: `HistoryAuthCtx` - 认证上下文
  - `limit`: `number` - 每页数量（默认100）
- **返回值**: `Promise<HistoryPage | null>`
- **关键逻辑**: 使用 `anchor_to_latest=true` 参数锚定到最新事件

#### `fetchOlderEvents`
- **类型**: `async function`
- **用途**: 获取指定ID之前的历史事件（更早的事件）
- **参数**: 
  - `ctx`: `HistoryAuthCtx` - 认证上下文
  - `beforeId`: `string` - 游标ID（获取此ID之前的事件）
  - `limit`: `number` - 每页数量（默认100）
- **返回值**: `Promise<HistoryPage | null>`
- **关键逻辑**: 使用 `before_id` 参数进行分页回溯

---

## 设计要点

1. **分页策略**: 采用游标分页（cursor-based pagination），而非偏移量分页，适合时间序列数据
2. **认证复用**: `HistoryAuthCtx` 设计允许认证信息一次性准备，多次复用于分页请求
3. **错误处理**: 静默失败模式，API错误返回null而非抛出异常，由调用方处理
4. **Beta特性**: 使用特定的Beta标识头访问实验性API

---

## 与其他文件的关系

- **依赖**:
  - `constants/oauth.js` - OAuth配置
  - `entrypoints/agentSdkTypes.js` - 消息类型定义
  - `utils/debug.js` - 调试工具
  - `utils/teleport/api.js` - API认证工具
- **被依赖**: 推测由会话历史UI组件或会话恢复功能使用

---

## 注意事项

1. **API稳定性**: 使用了Beta特性标识 `ccr-byoc-2025-07-29`，API可能不稳定
2. **超时设置**: 固定15秒超时，网络慢时可能失败
3. **错误静默**: 错误仅通过调试日志记录，调用方需检查null返回值
