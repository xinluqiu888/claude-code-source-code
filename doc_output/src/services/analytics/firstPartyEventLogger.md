# firstPartyEventLogger.ts — 第一方事件日志服务

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/analytics/firstPartyEventLogger.ts`
- **所属模块**: Analytics Service
- **功能类型**: 遥测后端集成

## 功能概述

该模块实现 Claude Code 的第一方（1P）事件日志系统，使用 OpenTelemetry SDK 将事件批量导出到 `/api/event_logging/batch` 端点。与 Datadog 不同，该系统支持更丰富的元数据和内部分析。

## 核心内容详解

### 采样配置

#### `EventSamplingConfig`
事件采样配置类型，支持按事件名称设置采样率（0-1）。

#### `getEventSamplingConfig(): EventSamplingConfig`
从 GrowthBook 获取事件采样配置。

#### `shouldSampleEvent(eventName): number | null`
确定事件是否应被采样：
- 返回 `null` — 未采样或 100% 采样
- 返回 `0` — 完全丢弃
- 返回采样率 — 事件被采样，采样率加入元数据

### 批处理配置

从 GrowthBook 动态配置 `tengu_1p_event_batch_config` 读取：
- `scheduledDelayMillis` — 导出间隔（默认 10s）
- `maxExportBatchSize` — 最大批次大小（默认 200）
- `maxQueueSize` — 最大队列大小（默认 8192）
- `skipAuth` — 是否跳过认证
- `maxAttempts` — 最大重试次数

### 主要函数

#### `initialize1PEventLogging(): void`
初始化 1P 事件日志基础设施。

**流程：**
1. 检查分析是否启用
2. 获取批处理配置
3. 创建资源属性（服务名、版本、平台信息）
4. 创建 `FirstPartyEventLoggingExporter`
5. 配置 `BatchLogRecordProcessor`
6. 初始化事件日志记录器

#### `logEventTo1P(eventName, metadata): void`
记录 1P 事件（同步入口）。
- 异步丰富元数据
- 包含核心元数据、用户元数据、事件元数据
- Fire-and-forget 模式，不阻塞

#### `logGrowthBookExperimentTo1P(data): void`
记录 GrowthBook 实验分配事件。

#### `reinitialize1PEventLoggingIfConfigChanged(): Promise<void>`
如果批处理配置变更，重建 1P 事件日志管道。

**事件丢失安全机制：**
1. 首先将日志记录器置空，新调用会失败保护
2. `forceFlush()` 排空旧处理器缓冲区
3. 导出失败的事件写入磁盘，新导出器重试
4. 交换到新提供者和日志记录器

#### `shutdown1PEventLogging(): Promise<void>`
优雅关闭 1P 事件日志记录器。

### OpenTelemetry 集成

```
LoggerProvider → BatchLogRecordProcessor → FirstPartyEventLoggingExporter
                      ↓
              定时/批次大小触发 → HTTP POST /api/event_logging/batch
```

## 设计要点

1. **独立管道** — 与客户 OTLP 遥测完全分离，防止数据泄漏
2. **磁盘备份** — 导出失败的事件写入磁盘，支持后续重试
3. **动态配置** — 支持运行时重建，响应配置变更
4. **元数据丰富** — 自动包含模型、会话、环境上下文等
5. **GrowthBook 集成** — 实验分配事件自动记录

## 与其他文件的关系

- **被调用**: `sink.ts` 的路由逻辑
- **依赖**: `firstPartyEventLoggingExporter.ts` 实现导出
- **关联**: `growthbook.ts` 获取配置和记录实验

## 注意事项

- 1P 事件日志与 Datadog 的主要区别：
  - 支持更丰富的元数据结构
  - 保留 `_PROTO_*` 键（PII 标记数据）
  - 使用 OpenTelemetry SDK 而非直接 HTTP
- 配置变更时会重建整个管道，期间可能丢失少量事件
- 磁盘备份路径：`~/.claude/telemetry/1p_failed_events.{sessionId}.{batchUuid}.json`
