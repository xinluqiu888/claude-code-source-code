# firstPartyEventLoggingExporter.ts — 第一方事件导出器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/analytics/firstPartyEventLoggingExporter.ts`
- **所属模块**: Analytics Service
- **功能类型**: OpenTelemetry LogRecordExporter 实现

## 功能概述

该模块实现 `LogRecordExporter` 接口，将 OpenTelemetry 日志记录导出到 `/api/event_logging/batch` API 端点。提供强大的错误恢复机制，包括磁盘持久化和指数退避重试。

## 核心内容详解

### 配置选项

```typescript
{
  timeout?: number              // 请求超时（默认 10s）
  maxBatchSize?: number         // 最大批次大小（默认 200）
  skipAuth?: boolean           // 跳过认证
  batchDelayMs?: number        // 批次间延迟（默认 100ms）
  baseBackoffDelayMs?: number  // 基础退避延迟（默认 500ms）
  maxBackoffDelayMs?: number   // 最大退避延迟（默认 30s）
  maxAttempts?: number         // 最大重试次数（默认 8）
  path?: string               // API 路径
  baseUrl?: string            // API 基础 URL
  isKilled?: () => boolean    // killswitch 检查
  schedule?: Function         // 调度函数（用于测试）
}
```

### 事件存储

#### 失败事件存储
- **位置**: `~/.claude/telemetry/1p_failed_events.{sessionId}.{batchUuid}.json`
- **格式**: JSON Lines（每行一个事件）
- **原子性**: 追加操作在大多数文件系统上是原子的

### 主要方法

#### `export(logs, resultCallback): Promise<void>`
OpenTelemetry 导出接口实现。

**流程：**
1. 检查是否已关闭
2. 过滤事件日志（按 scope name）
3. 转换日志为事件格式
4. 分批发送
5. 失败事件入队并调度退避重试

#### `sendEventsInBatches(events): Promise<FirstPartyEventLoggingEvent[]>`
将事件分块并发送。
- 首批次失败时短路，剩余批次直接入队
- 批次间延迟避免突发流量
- 返回失败事件列表

#### `sendBatchWithRetry(payload): Promise<void>`
带重试的单批次发送。

**认证策略：**
1. 首先尝试带认证发送
2. 401 错误时无认证重试
3. 无信任对话框时跳过认证
4. OAuth 令牌过期时跳过认证

#### `retryFailedEvents(): Promise<void>`
退避重试失败事件。
- 二次退避：base * attempts²
- 最大尝试次数后删除事件
- 成功时重置退避并继续重试

#### `retryPreviousBatches(): Promise<void>`
启动时重试之前会话的失败事件。
- 查找所有 `{sessionId}.*.json` 文件
- 排除当前批次 UUID
- 后台异步重试

### 事件转换

#### ClaudeCodeInternalEvent 转换
从 OpenTelemetry 日志属性中提取：
- `event_name` / `body` — 事件名称
- `core_metadata` — 核心元数据
- `user_metadata` — 用户元数据
- `event_metadata` — 事件特定元数据

转换为 Protocol Buffer JSON 格式，包含：
- `event_id` — 事件 UUID
- `event_name` — 事件名称
- `client_timestamp` — 客户端时间戳
- `session_id` — 会话 ID
- `device_id` — 设备 ID
- `auth` — 认证信息
- 核心字段（snake_case）
- `env` — 环境元数据
- `process` — 进程指标（base64）
- `skill_name` / `plugin_name` / `marketplace_name` — PII 标记字段
- `additional_metadata` — 额外元数据（base64 JSON）

#### GrowthbookExperimentEvent 转换
实验分配事件单独处理，包含：
- `experiment_id` — 实验 ID
- `variation_id` — 变体 ID
- `user_attributes` — 用户属性
- `experiment_metadata` — 实验元数据

### 退避策略

```
延迟 = min(baseBackoffDelayMs * attempts², maxBackoffDelayMs)

尝试 1: 500ms
尝试 2: 2000ms
尝试 3: 4500ms
...
尝试 8: 32000ms (封顶)
```

## 设计要点

1. **磁盘持久化** — 失败事件写入磁盘，会话间不丢失
2. **指数退避** — 避免服务端过载
3. **认证回退** — 401 错误时尝试无认证发送
4. **批量优化** — 支持大事件集的分块处理
5. **短路机制** — 首批次失败时停止后续批次

## 与其他文件的关系

- **被调用**: `firstPartyEventLogger.ts` 的初始化逻辑
- **依赖**: `metadata.ts` 的 `to1PEventFormat()` 转换
- **关联**: `index.ts` 的 `stripProtoFields()` 清理

## 注意事项

- 批次 UUID 隔离不同进程运行的事件文件
- `_PROTO_*` 键提升到顶层 proto 字段后从 additional_metadata 剥离
- 进程指标使用 base64 编码存储
- 仅处理 scope name 为 `com.anthropic.claude_code.events` 的日志
- ant 用户有额外的调试日志输出
