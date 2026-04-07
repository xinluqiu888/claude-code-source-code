# HybridTransport.ts — WebSocket读取与HTTP POST写入的混合传输层

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/transports/HybridTransport.ts` |
| **文件类型** | TypeScript 类文件 |
| **总行数** | 283 行 |
| **主要职责** | 实现混合传输模式：使用 WebSocket 接收服务端数据，使用 HTTP POST 向服务端发送数据 |

## 功能概述

`HybridTransport` 是一种混合传输实现，结合了 WebSocket 的读取能力和 HTTP POST 的写入能力。这种设计模式解决了在高并发场景下 WebSocket 写入可能导致的 Firestore 文档写入冲突问题。

在传统的 WebSocket 传输中，并发写入会导致并发 Firestore 写入到同一文档，产生冲突、重试风暴，最终可能导致页面级故障。`HybridTransport` 通过将写入操作序列化为单个 HTTP POST 请求，避免了这种竞争条件。

该传输层使用 `SerialBatchEventUploader` 来处理写入的序列化、批处理和重试逻辑。`stream_event` 类型的消息会被缓冲最多 100ms，以减少高频率内容增量更新时的 POST 请求数量。非流式写入会首先刷新缓冲的流事件，以保证消息顺序。

## 核心内容详解

### 导入依赖

```typescript
import axios, { type AxiosError } from 'axios'
import type { StdoutMessage } from 'src/entrypoints/sdk/controlTypes.js'
import { logForDebugging } from '../../utils/debug.js'
import { logForDiagnosticsNoPII } from '../../utils/diagLogs.js'
import { getSessionIngressAuthToken } from '../../utils/sessionIngressAuth.js'
import { SerialBatchEventUploader } from './SerialBatchEventUploader.js'
import { WebSocketTransport, type WebSocketTransportOptions } from './WebSocketTransport.js'
```

### 常量定义

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `BATCH_FLUSH_INTERVAL_MS` | 100 | `stream_event` 消息的批处理刷新间隔 |
| `POST_TIMEOUT_MS` | 15,000 | 单次 POST 请求超时时间 |
| `CLOSE_GRACE_MS` | 3,000 | 关闭时的优雅等待期 |

### 类定义

```typescript
export class HybridTransport extends WebSocketTransport {
  private postUrl: string
  private uploader: SerialBatchEventUploader<StdoutMessage>
  private streamEventBuffer: StdoutMessage[] = []
  private streamEventTimer: ReturnType<typeof setTimeout> | null = null
  
  constructor(
    url: URL,
    headers: Record<string, string> = {},
    sessionId?: string,
    refreshHeaders?: () => Record<string, string>,
    options?: WebSocketTransportOptions & {
      maxConsecutiveFailures?: number
      onBatchDropped?: (batchSize: number, failures: number) => void
    },
  )
}
```

### 核心方法

| 方法名 | 签名 | 功能说明 |
|--------|------|----------|
| `write` | `async write(message: StdoutMessage): Promise<void>` | 写入消息，`stream_event` 类型消息会被缓冲，其他类型立即发送 |
| `writeBatch` | `async writeBatch(messages: StdoutMessage[]): Promise<void>` | 批量写入消息 |
| `flush` | `flush(): Promise<void>` | 等待所有待处理事件完成 POST |
| `close` | `close(): void` | 关闭传输层，等待优雅关闭期后清理资源 |
| `droppedBatchCount` | `get droppedBatchCount(): number` | 获取因失败而丢弃的批次数量 |

### 私有方法

| 方法名 | 功能说明 |
|--------|----------|
| `postOnce` | 执行单次 HTTP POST 请求，处理重试逻辑和状态码判断 |
| `takeStreamEvents` | 获取并清空缓冲的流事件，清除定时器 |
| `flushStreamEvents` | 刷新流事件缓冲区到上传队列 |
| `convertWsUrlToPostUrl` | 将 WebSocket URL 转换为 HTTP POST 端点 URL |

## 设计要点

### 1. 序列化写入
通过 `SerialBatchEventUploader` 确保任意时刻只有一个 POST 请求在飞行中，避免并发冲突。

### 2. 批处理优化
`stream_event` 消息（通常是内容增量更新）会被缓冲 100ms，将多个小更新合并为单个 POST 请求，显著减少网络开销。

### 3. 重试策略
- 指数退避：基础延迟 500ms，最大延迟 8s
- 随机抖动：±1000ms 防止惊群效应
- 4xx 错误（除 429 外）视为永久性错误，直接丢弃
- 429 和 5xx 错误触发重试

### 4. 背压机制
当队列长度超过 `maxQueueSize`（100,000）时，`enqueue()` 会阻塞，给调用方提供背压信号。

### 5. 优雅关闭
关闭时给予 3s 的优雅期，允许队列中的消息尽可能完成发送。

## 与其他文件的关系

| 文件 | 关系说明 |
|------|----------|
| `WebSocketTransport.ts` | 继承基类，复用 WebSocket 读取能力 |
| `SerialBatchEventUploader.ts` | 依赖其处理写入的序列化、批处理和重试 |
| `transportUtils.ts` | 当 `CLAUDE_CODE_POST_FOR_SESSION_INGRESS_V2` 环境变量设置时使用 HybridTransport |
| `sessionIngressAuth.ts` | 使用 `getSessionIngressAuthToken()` 获取认证令牌 |

### 调用关系图

```
transportUtils.ts
    └── getTransportForUrl()
        └── HybridTransport (当 CLAUDE_CODE_POST_FOR_SESSION_INGRESS_V2 设置时)
            ├── extends WebSocketTransport (读取)
            └── uses SerialBatchEventUploader (写入)
```

## 注意事项

1. **消息顺序保证**：`stream_event` 消息缓冲后，非流式写入会首先刷新缓冲区，确保消息顺序正确。

2. **UUID 要求**：只有包含 `uuid` 属性的消息会被缓冲用于重放，无 UUID 的消息直接发送。

3. **认证依赖**：写入操作需要有效的 session token，否则消息会被静默丢弃。

4. **资源清理**：`close()` 方法会清理定时器和缓冲区，但需要在调用前显式 `flush()` 以确保消息送达。

5. **批处理大小**：最大批处理大小为 500 条消息，超出时会自动分片。
