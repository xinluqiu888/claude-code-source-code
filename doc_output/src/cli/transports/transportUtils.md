# transportUtils.ts — 传输层工具函数

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/transports/transportUtils.ts` |
| **文件类型** | TypeScript 工具文件 |
| **总行数** | 46 行 |
| **主要职责** | 根据 URL 和环境变量选择合适的传输层实现 |

## 功能概述

`transportUtils.ts` 提供了一个简单的工厂函数 `getTransportForUrl()`，用于根据配置 URL 和环境变量选择合适的传输层实现。这是 CLI 传输层选择的核心入口点。

支持的传输层选择优先级：
1. **SSETransport** (SSE 读取 + POST 写入)：当 `CLAUDE_CODE_USE_CCR_V2` 设置时
2. **HybridTransport** (WS 读取 + POST 写入)：当 `CLAUDE_CODE_POST_FOR_SESSION_INGRESS_V2` 设置时
3. **WebSocketTransport** (WS 读取 + WS 写入)：默认选项

该模块封装了传输层选择逻辑，使调用方无需关心具体的传输实现细节。

## 核心内容详解

### 导入依赖

```typescript
import { URL } from 'url'
import { isEnvTruthy } from '../../utils/envUtils.js'
import { HybridTransport } from './HybridTransport.js'
import { SSETransport } from './SSETransport.js'
import type { Transport } from './Transport.js'
import { WebSocketTransport } from './WebSocketTransport.js'
```

### 核心函数

```typescript
export function getTransportForUrl(
  url: URL,
  headers: Record<string, string> = {},
  sessionId?: string,
  refreshHeaders?: () => Record<string, string>,
): Transport
```

### 传输层选择逻辑

```typescript
if (isEnvTruthy(process.env.CLAUDE_CODE_USE_CCR_V2)) {
  // v2: SSE for reads, HTTP POST for writes
  const sseUrl = new URL(url.href)
  // 协议转换: wss: → https:, ws: → http:
  if (sseUrl.protocol === 'wss:') {
    sseUrl.protocol = 'https:'
  } else if (sseUrl.protocol === 'ws:') {
    sseUrl.protocol = 'http:'
  }
  // 构建 SSE 流 URL: .../sessions/{id}/worker/events/stream
  sseUrl.pathname = sseUrl.pathname.replace(/\/$/, '') + '/worker/events/stream'
  return new SSETransport(sseUrl, headers, sessionId, refreshHeaders)
}

if (url.protocol === 'ws:' || url.protocol === 'wss:') {
  if (isEnvTruthy(process.env.CLAUDE_CODE_POST_FOR_SESSION_INGRESS_V2)) {
    return new HybridTransport(url, headers, sessionId, refreshHeaders)
  }
  return new WebSocketTransport(url, headers, sessionId, refreshHeaders)
}
```

## 设计要点

### 1. 环境变量控制

| 环境变量 | 作用 |
|----------|------|
| `CLAUDE_CODE_USE_CCR_V2` | 启用 CCR v2 模式，使用 SSE + POST |
| `CLAUDE_CODE_POST_FOR_SESSION_INGRESS_V2` | 启用混合传输模式，使用 WS + POST |

两个环境变量互斥，优先级为 CCR_V2 > POST_V2 > 默认 WS。

### 2. URL 协议转换

当使用 SSETransport 时，自动将 WebSocket URL 转换为 HTTP URL：
- `wss://` → `https://`
- `ws://` → `http://`

路径自动追加 `/worker/events/stream` 构建 SSE 端点。

### 3. 统一接口

所有传输层实现相同的 `Transport` 接口，调用方可无缝切换：

```typescript
interface Transport {
  connect(): Promise<void>
  write(message: StdoutMessage): Promise<void>
  close(): void
  isConnectedStatus(): boolean
  isClosedStatus(): boolean
  setOnData(callback: (data: string) => void): void
  setOnClose(callback: (closeCode?: number) => void): void
}
```

## 与其他文件的关系

| 文件 | 关系说明 |
|------|----------|
| `HybridTransport.ts` | 在 POST_FOR_SESSION_INGRESS_V2 模式下实例化 |
| `SSETransport.ts` | 在 USE_CCR_V2 模式下实例化 |
| `WebSocketTransport.ts` | 默认模式下实例化 |
| `envUtils.ts` | 使用 `isEnvTruthy` 检查环境变量 |

### 调用关系图

```
print.ts / remoteIO.ts / bridge
    └── getTransportForUrl(url, headers, sessionId, refreshHeaders)
        ├── CLAUDE_CODE_USE_CCR_V2 → SSETransport
        ├── CLAUDE_CODE_POST_FOR_SESSION_INGRESS_V2 → HybridTransport
        └── default → WebSocketTransport
```

## 注意事项

1. **协议限制**：只支持 `ws:`, `wss:`, `http:`, `https:` 协议，其他协议会抛出错误。

2. **URL 构建**：CCR v2 模式下，SSE URL 从 session URL 派生，自动添加 `/worker/events/stream` 路径。

3. **Header 刷新**：`refreshHeaders` 回调用于连接断开时刷新认证头部。

4. **运行时检测**：`isEnvTruthy` 支持多种真值表示（`'1'`, `'true'`, `'yes'` 等）。

5. **返回值**：始终返回 `Transport` 接口的实现，调用方可安全地按接口使用。
