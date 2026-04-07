# SerialBatchEventUploader.ts — 串行批量事件上传器

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/transports/SerialBatchEventUploader.ts` |
| **文件类型** | TypeScript 类文件 |
| **总行数** | 276 行 |
| **主要职责** | 实现带批处理、重试和背压控制的串行事件上传 |

## 功能概述

`SerialBatchEventUploader` 是一个通用的串行批量事件上传器，设计用于保证事件按顺序可靠传输。它确保任意时刻只有一个 HTTP POST 请求在飞行中，避免了并发写入导致的竞争条件。

核心特性：
- **串行处理**：最多一个 POST 请求在飞行中
- **批处理**：每次请求最多处理 `maxBatchSize` 个事件
- **自动重试**：失败时指数退避重试，支持无限重试或最大失败次数限制
- **背压控制**：队列满时 `enqueue()` 阻塞，防止内存无限增长
- **优雅关闭**：支持 `flush()` 等待队列清空和 `close()` 立即丢弃

该组件被 `HybridTransport`、`ccrClient.ts` 等多个传输层使用，是可靠事件传输的基础构建块。

## 核心内容详解

### 导入依赖

```typescript
import { jsonStringify } from '../../utils/slowOperations.js'
```

### 异常类定义

```typescript
export class RetryableError extends Error {
  constructor(
    message: string,
    readonly retryAfterMs?: number,
  )
}
```

当 `send()` 抛出此错误时，上传器会使用 `retryAfterMs` 指定的等待时间（如 429 响应的 Retry-After 头部），而非指数退避计算。

### 配置类型

```typescript
type SerialBatchEventUploaderConfig<T> = {
  maxBatchSize: number           // 每批最大事件数
  maxBatchBytes?: number         // 每批最大字节数（可选）
  maxQueueSize: number           // 队列最大长度
  send: (batch: T[]) => Promise<void>  // 实际发送函数
  baseDelayMs: number            // 基础延迟
  maxDelayMs: number             // 最大延迟
  jitterMs: number               // 抖动范围
  maxConsecutiveFailures?: number // 最大连续失败次数（可选）
  onBatchDropped?: (batchSize: number, failures: number) => void // 丢弃回调
}
```

### 类定义

```typescript
export class SerialBatchEventUploader<T> {
  private pending: T[] = []
  private draining = false
  private closed = false
  private backpressureResolvers: Array<() => void> = []
  private flushResolvers: Array<() => void> = []
  private droppedBatches = 0
  
  constructor(config: SerialBatchEventUploaderConfig<T>)
}
```

### 核心方法

| 方法名 | 签名 | 功能说明 |
|--------|------|----------|
| `enqueue` | `async enqueue(events: T \| T[]): Promise<void>` | 添加事件到队列，队列满时阻塞 |
| `flush` | `flush(): Promise<void>` | 等待队列清空 |
| `close` | `close(): void` | 关闭上传器，丢弃待处理事件 |
| `droppedBatchCount` | `get droppedBatchCount(): number` | 获取丢弃的批次数 |
| `pendingCount` | `get pendingCount(): number` | 获取待处理事件数 |

### 私有方法

| 方法名 | 功能说明 |
|--------|----------|
| `drain` | 主循环，处理队列中的事件批次 |
| `takeBatch` | 从队列中提取一批事件，支持字节数限制 |
| `retryDelay` | 计算重试延迟，支持指数退避和抖动 |
| `releaseBackpressure` | 释放背压等待者 |
| `sleep` | 可中断的睡眠，支持提前唤醒 |

## 设计要点

### 1. 串行化保证

```typescript
private async drain(): Promise<void> {
  if (this.draining || this.closed) return
  this.draining = true  // 锁保证单实例运行
  // ...
}
```

`draining` 标志确保任意时刻只有一个 `drain` 实例在运行，保证事件处理的串行性。

### 2. 批处理策略

- 按 `maxBatchSize` 限制批次数量
- 可选按 `maxBatchBytes` 限制批次字节数
- 首个事件总是包含（即使超出字节限制）
- 不可序列化事件（如 BigInt、循环引用）会被就地丢弃

### 3. 重试机制

```typescript
private retryDelay(failures: number, retryAfterMs?: number): number {
  const jitter = Math.random() * this.config.jitterMs
  if (retryAfterMs !== undefined) {
    const clamped = Math.max(
      this.config.baseDelayMs,
      Math.min(retryAfterMs, this.config.maxDelayMs),
    )
    return clamped + jitter
  }
  const exponential = Math.min(
    this.config.baseDelayMs * 2 ** (failures - 1),
    this.config.maxDelayMs,
  )
  return exponential + jitter
}
```

- 支持服务器指定的重试延迟（`RetryableError.retryAfterMs`）
- 指数退避：基础延迟 × 2^(失败次数-1)
- 随机抖动避免惊群效应
- 可配置最大连续失败次数，超过则丢弃批次

### 4. 背压控制

```typescript
while (
  this.pending.length + items.length > this.config.maxQueueSize &&
  !this.closed
) {
  await new Promise<void>(resolve => {
    this.backpressureResolvers.push(resolve)
  })
}
```

当队列满时，`enqueue()` 阻塞直到有空间。`releaseBackpressure()` 在成功发送后释放等待者。

### 5. 错误处理策略

- 失败批次重新入队（使用 `concat` 避免 `unshift` 的性能问题）
- 支持 `maxConsecutiveFailures` 限制，防止无限重试
- 丢弃时调用 `onBatchDropped` 回调通知调用方

## 与其他文件的关系

| 文件 | 关系说明 |
|------|----------|
| `HybridTransport.ts` | 使用 `SerialBatchEventUploader<StdoutMessage>` 处理写入 |
| `ccrClient.ts` | 使用多个上传器实例处理不同类型的事件 |
| `WorkerStateUploader.ts` | 类似但更简单的状态上传器 |
| `slowOperations.ts` | 使用 `jsonStringify` 序列化事件 |

### 调用关系图

```
HybridTransport
    └── SerialBatchEventUploader<StdoutMessage>
        └── send(batch) → HTTP POST
            └── on failure → retry with backoff

ccrClient.ts
    ├── eventUploader: SerialBatchEventUploader<ClientEvent>
    ├── internalEventUploader: SerialBatchEventUploader<WorkerEvent>
    └── deliveryUploader: SerialBatchEventUploader<DeliveryUpdate>
```

## 注意事项

1. **内存限制**：`maxQueueSize` 应设置得足够高以作为内存限制，实际背压应由调用方控制。

2. **批处理字节限制**：当设置 `maxBatchBytes` 时，首个事件总是包含，即使超出限制。

3. **失败批次重入队**：使用 `batch.concat(this.pending)` 而非 `unshift(...batch)`，避免每次失败都移动所有待处理项。

4. **优雅关闭**：`close()` 会立即清空队列并解析所有等待者，不会尝试发送剩余事件。

5. **重试延迟抖动**：抖动范围是 `[0, jitterMs)`，实际延迟为 `exponential + jitter`。

6. **不可序列化事件处理**：在 `takeBatch` 中捕获序列化异常并就地删除问题事件，防止队列中毒。
