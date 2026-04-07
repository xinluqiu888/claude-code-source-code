# WebSocketTransport.ts — WebSocket 传输实现

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/transports/WebSocketTransport.ts` |
| **文件类型** | TypeScript 类文件 |
| **总行数** | 801 行 |
| **主要职责** | 实现基于 WebSocket 的双向通信传输层，支持自动重连和消息缓冲 |

## 功能概述

`WebSocketTransport` 是一个完整的 WebSocket 传输实现，支持 Node.js 和 Bun 运行时。它提供了可靠的 WebSocket 通信能力，包括自动重连、消息缓冲重放、连接健康检测等功能。

核心特性：
- **跨运行时支持**：自动检测并适配 Bun 和 Node.js (`ws` 包)
- **自动重连**：指数退避策略，最多尝试 10 分钟
- **消息缓冲**：断线期间消息缓冲，重连后重放
- **健康检测**：定期 ping/pong 检测连接活性
- **代理支持**：支持 HTTP/HTTPS 代理
- **睡眠检测**：检测系统睡眠/唤醒，重置重连预算

该传输层是 Claude Code CLI 的基础通信组件，被 `HybridTransport` 继承用于读取端。

## 核心内容详解

### 导入依赖

```typescript
import type { StdoutMessage } from 'src/entrypoints/sdk/controlTypes.js'
import type WsWebSocket from 'ws'
import { logEvent } from '../../services/analytics/index.js'
import { CircularBuffer } from '../../utils/CircularBuffer.js'
import { logForDebugging } from '../../utils/debug.js'
import { logForDiagnosticsNoPII } from '../../utils/diagLogs.js'
import { isEnvTruthy } from '../../utils/envUtils.js'
import { getWebSocketTLSOptions } from '../../utils/mtls.js'
import { getWebSocketProxyAgent, getWebSocketProxyUrl } from '../../utils/proxy.js'
import { registerSessionActivityCallback, unregisterSessionActivityCallback } from '../../utils/sessionActivity.js'
import { jsonStringify } from '../../utils/slowOperations.js'
import type { Transport } from './Transport.js'
```

### 常量定义

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `KEEP_ALIVE_FRAME` | `'{"type":"keep_alive"}\n'` | keepalive 数据帧 |
| `DEFAULT_MAX_BUFFER_SIZE` | 1,000 | 消息缓冲区最大大小 |
| `DEFAULT_BASE_RECONNECT_DELAY` | 1,000 | 重连基础延迟 |
| `DEFAULT_MAX_RECONNECT_DELAY` | 30,000 | 重连最大延迟 |
| `DEFAULT_RECONNECT_GIVE_UP_MS` | 600,000 | 重连时间预算 |
| `DEFAULT_PING_INTERVAL` | 10,000 | ping 间隔 |
| `DEFAULT_KEEPALIVE_INTERVAL` | 300,000 | keepalive 间隔（5分钟） |
| `SLEEP_DETECTION_THRESHOLD_MS` | 60,000 | 睡眠检测阈值 |

### 类型定义

```typescript
export type WebSocketTransportOptions = {
  autoReconnect?: boolean  // 是否自动重连（默认 true）
  isBridge?: boolean       // 是否桥接模式（控制遥测）
}

type WebSocketTransportState = 'idle' | 'connected' | 'reconnecting' | 'closing' | 'closed'

// 通用 WebSocket 接口抽象
type WebSocketLike = {
  close(): void
  send(data: string): void
  ping?(): void
}
```

### 类定义

```typescript
export class WebSocketTransport implements Transport {
  protected url: URL
  protected state: WebSocketTransportState = 'idle'
  private ws: WebSocketLike | null = null
  private messageBuffer: CircularBuffer<StdoutMessage>
  private reconnectAttempts = 0
  private reconnectStartTime: number | null = null
  private pingInterval: NodeJS.Timeout | null = null
  private keepAliveInterval: NodeJS.Timeout | null = null
  private lastActivityTime = 0
  // ... 其他属性
}
```

### 核心方法

| 方法名 | 签名 | 功能说明 |
|--------|------|----------|
| `connect` | `async connect(): Promise<void>` | 建立 WebSocket 连接 |
| `write` | `async write(message: StdoutMessage): Promise<void>` | 发送消息（带缓冲） |
| `close` | `close(): void` | 关闭连接 |
| `isConnectedStatus` | `isConnectedStatus(): boolean` | 检查是否已连接 |
| `isClosedStatus` | `isClosedStatus(): boolean` | 检查是否已关闭 |
| `setOnData` | `setOnData(callback: (data: string) => void): void` | 设置数据接收回调 |
| `setOnClose` | `setOnClose(callback: (closeCode?: number) => void): void` | 设置关闭回调 |

## 设计要点

### 1. 运行时适配

```typescript
if (typeof Bun !== 'undefined') {
  // Bun 原生 WebSocket
  const ws = new globalThis.WebSocket(this.url.href, { headers, proxy, tls })
  this.isBunWs = true
} else {
  // Node.js ws 包
  const { default: WS } = await import('ws')
  const ws = new WS(this.url.href, { headers, agent, ...tlsOptions })
  this.isBunWs = false
}
```

根据运行时自动选择 WebSocket 实现，并附加相应的事件监听器。

### 2. 事件处理器管理

```typescript
private removeWsListeners(ws: WebSocketLike): void {
  if (this.isBunWs) {
    // Bun: removeEventListener
    nws.removeEventListener('open', this.onBunOpen)
    // ...
  } else {
    // Node: off
    nws.off('open', this.onNodeOpen)
    // ...
  }
}
```

使用类属性箭头函数存储处理器，确保可以正确移除，避免内存泄漏。

### 3. 消息缓冲与重放

```typescript
async write(message: StdoutMessage): Promise<void> {
  if ('uuid' in message && typeof message.uuid === 'string') {
    this.messageBuffer.add(message)
    this.lastSentId = message.uuid
  }
  // ...
}

private replayBufferedMessages(lastId: string): void {
  // 根据服务器确认的 lastId 确定重放起始位置
  // 只重放未确认的消息
}
```

使用环形缓冲区存储消息，支持断线后重放未确认的消息。

### 4. 睡眠检测

```typescript
if (this.lastReconnectAttemptTime !== null &&
    now - this.lastReconnectAttemptTime > SLEEP_DETECTION_THRESHOLD_MS) {
  // 检测到睡眠，重置重连预算
  this.reconnectStartTime = now
  this.reconnectAttempts = 0
}
```

检测系统睡眠/唤醒（通过重连尝试间隔），重置重连预算以便立即重试。

### 5. 活性检测

```typescript
private startPingInterval(): void {
  this.pingInterval = setInterval(() => {
    // 检测 tick 间隔，判断进程是否被挂起
    if (gap > SLEEP_DETECTION_THRESHOLD_MS) {
      this.handleConnectionError()
      return
    }
    // 检查 pong 响应
    if (!this.pongReceived) {
      this.handleConnectionError()
      return
    }
    this.ws?.ping?.()
  }, DEFAULT_PING_INTERVAL)
}
```

定期发送 ping，检测连接活性，同时检测进程挂起（如笔记本合盖）。

## 与其他文件的关系

| 文件 | 关系说明 |
|------|----------|
| `HybridTransport.ts` | 继承 `WebSocketTransport`，复用其读取能力 |
| `transportUtils.ts` | 默认使用 `WebSocketTransport` 进行通信 |
| `sessionActivity.ts` | 注册活动回调，在会话活动时发送 keepalive |
| `proxy.ts` | 使用代理配置 |
| `mtls.ts` | 使用 TLS 配置 |

### 调用关系图

```
transportUtils.ts (默认)
    └── WebSocketTransport
        ├── connect() → WebSocket
        ├── write() → ws.send()
        └── auto-reconnect on disconnect

HybridTransport.ts
    └── extends WebSocketTransport (复用读取)
        └── overrides write() (使用 POST)
```

## 注意事项

1. **消息 UUID**：只有包含 `uuid` 属性的消息会被缓冲用于重放，确保消息可追踪。

2. **环形缓冲区**：使用 `CircularBuffer` 限制内存使用，旧消息会被覆盖。

3. **重连预算**：10 分钟重连预算后放弃，触发 `onClose` 回调。

4. **永久关闭码**：1002（协议错误）、4001（会话过期）、4003（未授权）不会触发重连。

5. **Token 刷新**：收到 4003 时尝试刷新头部，成功后继续重连。

6. **代理支持**：通过 `getWebSocketProxyAgent` 和 `getWebSocketProxyUrl` 支持代理。

7. **TLS 配置**：通过 `getWebSocketTLSOptions` 支持 mTLS。
