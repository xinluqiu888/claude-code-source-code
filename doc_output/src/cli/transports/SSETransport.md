# SSETransport.ts — Server-Sent Events 传输实现

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/transports/SSETransport.ts` |
| **文件类型** | TypeScript 类文件 |
| **总行数** | 712 行 |
| **主要职责** | 实现基于 Server-Sent Events (SSE) 的读取和 HTTP POST 写入传输层 |

## 功能概述

`SSETransport` 实现了完整的 SSE 传输协议，用于 CCR v2 事件流通信。它通过 SSE 连接从服务器接收事件流，并使用 HTTP POST 向服务器发送事件。

该传输层支持以下核心功能：
- **自动重连**：指数退避重连策略，最多尝试 10 分钟
- **事件恢复**：通过 `Last-Event-ID` 头部支持断点续传
- **活性检测**：45 秒超时检测连接是否存活
- **重复检测**：基于序列号的重复事件过滤
- **POST 重试**：写入操作支持指数退避重试

每个 `event: client_event` 帧携带 `StreamClientEvent` 协议的 JSON 数据，传输层提取 `payload` 并以换行分隔的 JSON 格式传递给消费者。

## 核心内容详解

### 导入依赖

```typescript
import axios, { type AxiosError } from 'axios'
import type { StdoutMessage } from 'src/entrypoints/sdk/controlTypes.js'
import { logForDebugging } from '../../utils/debug.js'
import { logForDiagnosticsNoPII } from '../../utils/diagLogs.js'
import { errorMessage } from '../../utils/errors.js'
import { getSessionIngressAuthHeaders } from '../../utils/sessionIngressAuth.js'
import { sleep } from '../../utils/sleep.js'
import { jsonParse, jsonStringify } from '../../utils/slowOperations.js'
import { getClaudeCodeUserAgent } from '../../utils/userAgent.js'
import type { Transport } from './Transport.js'
```

### 常量定义

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `RECONNECT_BASE_DELAY_MS` | 1,000 | 重连基础延迟 |
| `RECONNECT_MAX_DELAY_MS` | 30,000 | 重连最大延迟 |
| `RECONNECT_GIVE_UP_MS` | 600,000 | 重连时间预算（10分钟） |
| `LIVENESS_TIMEOUT_MS` | 45,000 | 活性检测超时 |
| `POST_MAX_RETRIES` | 10 | POST 最大重试次数 |
| `POST_BASE_DELAY_MS` | 500 | POST 基础延迟 |
| `POST_MAX_DELAY_MS` | 8,000 | POST 最大延迟 |

### 类型定义

```typescript
type SSETransportState = 'idle' | 'connected' | 'reconnecting' | 'closing' | 'closed'

export type StreamClientEvent = {
  event_id: string
  sequence_num: number
  event_type: string
  source: string
  payload: Record<string, unknown>
  created_at: string
}

type SSEFrame = {
  event?: string
  id?: string
  data?: string
}
```

### 类定义

```typescript
export class SSETransport implements Transport {
  private state: SSETransportState = 'idle'
  private lastSequenceNum = 0
  private seenSequenceNums = new Set<number>()
  private reconnectAttempts = 0
  private reconnectStartTime: number | null = null
  private abortController: AbortController | null = null
  // ... 其他属性和方法
}
```

### 核心方法

| 方法名 | 签名 | 功能说明 |
|--------|------|----------|
| `connect` | `async connect(): Promise<void>` | 建立 SSE 连接，支持断点续传 |
| `write` | `async write(message: StdoutMessage): Promise<void>` | 通过 HTTP POST 发送消息 |
| `close` | `close(): void` | 关闭连接，清理资源 |
| `getLastSequenceNum` | `getLastSequenceNum(): number` | 获取最后处理的序列号 |
| `isConnectedStatus` | `isConnectedStatus(): boolean` | 检查是否已连接 |
| `isClosedStatus` | `isClosedStatus(): boolean` | 检查是否已关闭 |

### 工具函数

```typescript
export function parseSSEFrames(buffer: string): { frames: SSEFrame[]; remaining: string }
```

解析 SSE 帧，支持增量解析（流式处理）。

## 设计要点

### 1. SSE 帧解析
遵循 SSE 规范解析帧：
- 双换行符 (`\n\n`) 分隔帧
- 字段格式：`field: value`
- 多个 `data:` 行用 `\n` 连接
- 支持注释行（以 `:` 开头）

### 2. 自动重连策略
- 基础延迟 1s，指数增长至最大 30s
- 随机抖动 ±25% 防止惊群效应
- 10 分钟时间预算后放弃重连
- 永久错误码（401, 403, 404）立即终止

### 3. 断点续传
- 使用 `Last-Event-ID` 头部请求从指定位置恢复
- 维护 `lastSequenceNum` 作为高水位标记
- 序列号去重防止重复处理

### 4. 活性检测
- 45 秒超时无帧则判定连接死亡
- 服务器每 15 秒发送 keepalive 注释
- 超时后触发重连

### 5. POST 写入重试
- 10 次最大重试
- 4xx 错误（除 429）视为永久错误
- 429/5xx 触发指数退避重试

## 与其他文件的关系

| 文件 | 关系说明 |
|------|----------|
| `Transport.ts` | 实现 Transport 接口 |
| `transportUtils.ts` | 当 `CLAUDE_CODE_USE_CCR_V2` 设置时使用 SSETransport |
| `ccrClient.ts` | 使用 SSETransport 进行 CCR v2 通信 |
| `sessionIngressAuth.ts` | 使用 `getSessionIngressAuthHeaders()` 获取认证头部 |

### 调用关系图

```
transportUtils.ts
    └── getTransportForUrl()
        └── SSETransport (当 CLAUDE_CODE_USE_CCR_V2 设置时)
            └── connect() → fetch() SSE stream
            └── write() → axios.post()
```

## 注意事项

1. **协议转换**：SSE URL 会自动从 WebSocket 协议转换（wss:→https:, ws:→http:）

2. **事件过滤**：只处理 `event: client_event` 类型的事件，其他类型会记录警告

3. **重复检测**：使用 `seenSequenceNums` Set 存储已见序列号，超过 1000 条时清理旧条目

4. **认证头部**：支持 Cookie 认证，当使用 Cookie 时会移除 Authorization 头部以避免混淆

5. **流式解析**：`parseSSEFrames` 支持增量解析，可处理不完整的帧缓冲

6. **线程安全**：所有状态变更都在单线程中处理，无需额外同步
