# WorkerStateUploader.ts — 工作器状态合并上传器

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/transports/WorkerStateUploader.ts` |
| **文件类型** | TypeScript 类文件 |
| **总行数** | 132 行 |
| **主要职责** | 实现工作器状态的合并上传，支持 PUT /worker 端点的状态报告 |

## 功能概述

`WorkerStateUploader` 是一个专门用于上传工作器状态的组件，用于向 CCR (Claude Code Remote) 的 PUT /worker 端点报告会话状态。它实现了智能的状态合并策略，以最小化网络请求次数。

核心特性：
- **请求合并**：新调用与待处理补丁合并，永远不超过 1 个待处理 + 1 个在途
- **补丁合并**：顶层键采用"最后值获胜"策略，元数据对象采用 RFC 7396 合并
- **自动重试**：指数退避重试直到成功或关闭
- **自然有界**：无需背压，天然限制在 2 个槽位（在途 + 待处理）

与 `SerialBatchEventUploader` 不同，`WorkerStateUploader` 针对单对象 PUT 操作优化，适合频繁更新的状态报告场景。

## 核心内容详解

### 导入依赖

```typescript
import { sleep } from '../../utils/sleep.js'
```

### 配置类型

```typescript
type WorkerStateUploaderConfig = {
  send: (body: Record<string, unknown>) => Promise<boolean>
  baseDelayMs: number   // 基础延迟
  maxDelayMs: number    // 最大延迟
  jitterMs: number      // 抖动范围
}
```

### 类定义

```typescript
export class WorkerStateUploader {
  private inflight: Promise<void> | null = null
  private pending: Record<string, unknown> | null = null
  private closed = false
  private readonly config: WorkerStateUploaderConfig

  constructor(config: WorkerStateUploaderConfig)
}
```

### 核心方法

| 方法名 | 签名 | 功能说明 |
|--------|------|----------|
| `enqueue` | `enqueue(patch: Record<string, unknown>): void` | 入队一个状态补丁，与现有待处理补丁合并 |
| `close` | `close(): void` | 关闭上传器，丢弃待处理补丁 |

### 私有方法

| 方法名 | 功能说明 |
|--------|----------|
| `drain` | 处理待发送的状态补丁 |
| `sendWithRetry` | 带指数退避重试的发送 |
| `retryDelay` | 计算重试延迟 |
| `coalescePatches` | 合并两个补丁（RFC 7396 风格） |

## 设计要点

### 1. 单槽合并策略

```typescript
enqueue(patch: Record<string, unknown>): void {
  if (this.closed) return
  this.pending = this.pending ? coalescePatches(this.pending, patch) : patch
  void this.drain()
}
```

新补丁与待处理补丁合并，永远只保留一个待处理状态，避免队列无限增长。

### 2. RFC 7396 合并

```typescript
function coalescePatches(
  base: Record<string, unknown>,
  overlay: Record<string, unknown>,
): Record<string, unknown> {
  for (const [key, value] of Object.entries(overlay)) {
    if (
      (key === 'external_metadata' || key === 'internal_metadata') &&
      merged[key] && typeof merged[key] === 'object' &&
      typeof value === 'object' && value !== null
    ) {
      // RFC 7396 合并
      merged[key] = { ...(merged[key]), ...(value) }
    } else {
      // 最后值获胜
      merged[key] = value
    }
  }
}
```

- 顶层键：`overlay` 的值直接覆盖 `base`
- `external_metadata`/`internal_metadata`：对象深层合并，支持增量更新

### 3. 在途 + 待处理双槽

```typescript
private async drain(): Promise<void> {
  if (this.inflight || this.closed) return
  if (!this.pending) return

  const payload = this.pending
  this.pending = null

  this.inflight = this.sendWithRetry(payload).then(() => {
    this.inflight = null
    if (this.pending && !this.closed) {
      void this.drain()  // 有新待处理，继续发送
    }
  })
}
```

- 最多 1 个在途请求 + 1 个待处理补丁
- 发送完成后自动检查并发送新待处理

### 4. 重试期间的补丁吸收

```typescript
private async sendWithRetry(payload: Record<string, unknown>): Promise<void> {
  while (!this.closed) {
    const ok = await this.config.send(current)
    if (ok) return

    failures++
    await sleep(this.retryDelay(failures))

    // 吸收重试期间到达的新补丁
    if (this.pending && !this.closed) {
      current = coalescePatches(current, this.pending)
      this.pending = null
    }
  }
}
```

重试期间到达的新补丁会被合并到当前负载中，避免丢失更新。

## 与其他文件的关系

| 文件 | 关系说明 |
|------|----------|
| `ccrClient.ts` | 使用 `WorkerStateUploader` 报告工作器状态和元数据 |
| `sleep.ts` | 使用 `sleep` 实现重试延迟 |

### 调用关系图

```
ccrClient.ts
    └── workerState: WorkerStateUploader
        ├── reportState() → enqueue({ worker_status, requires_action_details })
        ├── reportMetadata() → enqueue({ external_metadata })
        └── send() → PUT /sessions/{id}/worker
```

## 注意事项

1. **Fire-and-forget**：`enqueue()` 返回 void，调用方无需等待，适合状态报告的异步特性。

2. **最后值获胜**：非元数据键的最新值会完全覆盖旧值，适合状态枚举值（如 `worker_status`）。

3. **元数据合并**：`external_metadata` 和 `internal_metadata` 支持增量更新，新键添加、旧键覆盖。

4. **关闭行为**：`close()` 立即丢弃待处理补丁，不尝试发送，适合快速关闭场景。

5. **无限重试**：默认无限重试直到成功，没有最大失败次数限制。

6. **抖动范围**：重试延迟为 `exponential + jitter`，`jitter` 在 `[0, jitterMs)` 范围内随机。
