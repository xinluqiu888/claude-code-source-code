# ccrClient.ts — Claude Code Remote (CCR) 客户端

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/transports/ccrClient.ts` |
| **文件类型** | TypeScript 类文件 |
| **总行数** | 999 行 |
| **主要职责** | 管理与 CCR 服务端的完整生命周期协议，包括状态报告、事件上传、心跳保活和会话恢复 |

## 功能概述

`CCRClient` 是 Claude Code CLI 与 CCR (Claude Code Remote) 服务端通信的核心客户端，实现了完整的 worker 生命周期协议。它封装了所有与 CCR v2 API 的交互，包括：

- **Worker 注册与初始化**：报告 worker 上线，设置 epoch
- **状态报告**：PUT /worker 端点报告 worker 状态和元数据
- **事件流管理**：client events（用户可见）和 internal events（内部状态）的双轨上传
- **心跳保活**：定期发送心跳维持会话活性
- **会话恢复**：读取 internal events 恢复会话状态
- **流事件优化**：text_delta 累积合并，减少网络请求

该客户端是 CCR v2 架构的核心组件，支持远程会话、分布式 worker 和会话持久化。

## 核心内容详解

### 导入依赖

```typescript
import { randomUUID } from 'crypto'
import type { SDKPartialAssistantMessage, StdoutMessage } from 'src/entrypoints/sdk/controlTypes.js'
import { decodeJwtExpiry } from '../../bridge/jwtUtils.js'
import { logForDebugging } from '../../utils/debug.js'
import { logForDiagnosticsNoPII } from '../../utils/diagLogs.js'
import { errorMessage, getErrnoCode } from '../../utils/errors.js'
import { createAxiosInstance } from '../../utils/proxy.js'
import { registerSessionActivityCallback, unregisterSessionActivityCallback } from '../../utils/sessionActivity.js'
import { getSessionIngressAuthHeaders, getSessionIngressAuthToken } from '../../utils/sessionIngressAuth.js'
import type { RequiresActionDetails, SessionState } from '../../utils/sessionState.js'
import { sleep } from '../../utils/sleep.js'
import { getClaudeCodeUserAgent } from '../../utils/userAgent.js'
import { RetryableError, SerialBatchEventUploader } from './SerialBatchEventUploader.js'
import type { SSETransport, StreamClientEvent } from './SSETransport.js'
import { WorkerStateUploader } from './WorkerStateUploader.js'
```

### 常量定义

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `DEFAULT_HEARTBEAT_INTERVAL_MS` | 20,000 | 默认心跳间隔 |
| `STREAM_EVENT_FLUSH_INTERVAL_MS` | 100 | 流事件刷新间隔 |
| `MAX_CONSECUTIVE_AUTH_FAILURES` | 10 | 最大连续认证失败次数 |

### 类型定义

```typescript
export type CCRInitFailReason =
  | 'no_auth_headers'
  | 'missing_epoch'
  | 'worker_register_failed'

export class CCRInitError extends Error {
  constructor(readonly reason: CCRInitFailReason)
}

export type StreamAccumulatorState = {
  byMessage: Map<string, string[][]>
  scopeToMessage: Map<string, string>
}

type EventPayload = { uuid: string; type: string; [key: string]: unknown }
type ClientEvent = { payload: EventPayload; ephemeral?: boolean }
type WorkerEvent = { payload: EventPayload; is_compaction?: boolean; agent_id?: string }
export type InternalEvent = { event_id: string; event_type: string; payload: Record<string, unknown>; ... }
```

### 类定义

```typescript
export class CCRClient {
  private workerEpoch = 0
  private heartbeatTimer: NodeJS.Timeout | null = null
  private consecutiveAuthFailures = 0
  private currentState: SessionState | null = null
  private sessionBaseUrl: string
  private sessionId: string

  // 上传器
  private workerState: WorkerStateUploader
  private eventUploader: SerialBatchEventUploader<ClientEvent>
  private internalEventUploader: SerialBatchEventUploader<WorkerEvent>
  private deliveryUploader: SerialBatchEventUploader<{ eventId: string; status: 'received' | 'processing' | 'processed' }>

  // 流事件优化
  private streamEventBuffer: SDKPartialAssistantMessage[] = []
  private streamEventTimer: ReturnType<typeof setTimeout> | null = null
  private streamTextAccumulator = createStreamAccumulator()

  constructor(
    transport: SSETransport,
    sessionUrl: URL,
    opts?: {
      onEpochMismatch?: () => never
      heartbeatIntervalMs?: number
      getAuthHeaders?: () => Record<string, string>
    }
  )
}
```

### 核心方法

| 方法名 | 签名 | 功能说明 |
|--------|------|----------|
| `initialize` | `async initialize(epoch?: number): Promise<Record<string, unknown> \| null>` | 初始化 worker，注册到 CCR |
| `reportState` | `reportState(state: SessionState, details?: RequiresActionDetails): void` | 报告 worker 状态 |
| `reportMetadata` | `reportMetadata(metadata: Record<string, unknown>): void` | 报告元数据 |
| `writeEvent` | `async writeEvent(message: StdoutMessage): Promise<void>` | 写入 client event |
| `writeInternalEvent` | `async writeInternalEvent(eventType: string, payload: Record<string, unknown>, opts?): Promise<void>` | 写入 internal event |
| `flush` | `async flush(): Promise<void>` | 刷新 client events |
| `flushInternalEvents` | `flushInternalEvents(): Promise<void>` | 刷新 internal events |
| `readInternalEvents` | `async readInternalEvents(): Promise<InternalEvent[] \| null>` | 读取 foreground agent events |
| `readSubagentInternalEvents` | `async readSubagentInternalEvents(): Promise<InternalEvent[] \| null>` | 读取 subagent events |
| `reportDelivery` | `reportDelivery(eventId: string, status: DeliveryStatus): void` | 报告事件投递状态 |
| `close` | `close(): void` | 关闭客户端 |

### 流事件累积函数

```typescript
export function createStreamAccumulator(): StreamAccumulatorState
export function accumulateStreamEvents(
  buffer: SDKPartialAssistantMessage[],
  state: StreamAccumulatorState,
): EventPayload[]
export function clearStreamAccumulatorForMessage(
  state: StreamAccumulatorState,
  assistant: { session_id: string; parent_tool_use_id: string | null; message: { id: string } },
): void
```

## 设计要点

### 1. 双轨事件系统

```
Client Events (用户可见)
    └── POST /sessions/{id}/worker/events
    └── 通过 SSE 流向前端

Internal Events (内部状态)
    └── POST /sessions/{id}/worker/internal-events
    └── 用于会话恢复，不向前端暴露
```

### 2. text_delta 累积优化

```typescript
export function accumulateStreamEvents(buffer, state): EventPayload[] {
  // 将同一 content block 的多个 text_delta 累积为完整快照
  // 每个 flush 发出一个包含完整文本的事件
  // 客户端中途连接时能看到完整内容，而非片段
}
```

### 3. 认证失败处理

```typescript
if (response.status === 401 || response.status === 403) {
  const exp = decodeJwtExpiry(token)
  if (exp !== null && exp * 1000 < Date.now()) {
    // JWT 已过期，立即退出
    this.onEpochMismatch()
  }
  // 令牌看起来有效但服务器返回 401
  this.consecutiveAuthFailures++
  if (this.consecutiveAuthFailures >= MAX_CONSECUTIVE_AUTH_FAILURES) {
    this.onEpochMismatch()
  }
}
```

- 检测到 JWT 过期立即退出
- 10 次连续认证失败后触发 epoch mismatch 处理

### 4. Epoch 冲突处理

```typescript
if (response.status === 409) {
  this.handleEpochMismatch()  // 新 worker 已替换此实例
}

private handleEpochMismatch(): never {
  this.onEpochMismatch()  // 默认 process.exit(1)
}
```

409 响应表示新 worker 实例已注册，当前实例应立即退出。

### 5. 会话状态恢复

```typescript
private async getWorkerState(): Promise<{ metadata; durationMs }> {
  // GET /sessions/{id}/worker
  // 读取之前 worker 写入的 external_metadata
}
```

启动时并发读取之前的 worker 状态，用于会话恢复。

## 与其他文件的关系

| 文件 | 关系说明 |
|------|----------|
| `SerialBatchEventUploader.ts` | 使用三个上传器实例分别处理 client/internal/delivery 事件 |
| `WorkerStateUploader.ts` | 使用状态上传器报告 worker 状态和元数据 |
| `SSETransport.ts` | 依赖 SSE 传输接收事件，并设置 onEvent 回调处理送达确认 |
| `sessionState.ts` | 使用 `SessionState` 和 `RequiresActionDetails` 类型 |
| `sessionIngressAuth.ts` | 使用认证函数获取请求头部 |
| `sessionActivity.ts` | 注册活动回调，在会话活动时发送 keep_alive |

### 调用关系图

```
remoteIO.ts / replBridge.ts
    └── CCRClient
        ├── initialize() → PUT /worker (worker_status: idle)
        ├── workerState.enqueue() → PUT /worker
        ├── eventUploader.enqueue() → POST /worker/events
        ├── internalEventUploader.enqueue() → POST /worker/internal-events
        ├── deliveryUploader.enqueue() → POST /worker/events/delivery
        ├── startHeartbeat() → POST /worker/heartbeat
        └── readInternalEvents() → GET /worker/internal-events
```

## 注意事项

1. **Epoch 管理**：`workerEpoch` 从环境变量 `CLAUDE_CODE_WORKER_EPOCH` 或构造函数参数获取，用于解决 worker 竞争条件。

2. **UUID 注入**：所有事件自动注入 UUID，确保服务端幂等性（重试不重复处理）。

3. **流事件缓冲**：`stream_event` 类型消息缓冲 100ms，期间累积的 text_delta 合并为完整快照。

4. **并发安全**：所有上传器都是串行的，CCRClient 本身不保证跨方法调用的原子性。

5. **关闭顺序**：`close()` 停止心跳、注销回调、关闭所有上传器，但不会等待上传完成。

6. **分页读取**：`readInternalEvents` 和 `readSubagentInternalEvents` 自动处理分页游标。

7. **Token 过期检测**：通过 `decodeJwtExpiry` 解析 JWT 过期时间，避免不必要的重试。
