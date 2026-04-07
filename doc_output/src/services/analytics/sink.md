# sink.ts — 分析 Sink 实现

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/analytics/sink.ts`
- **所属模块**: Analytics Service
- **功能类型**: 事件路由实现

## 功能概述

该模块包含分析路由逻辑的实际实现，在应用启动时初始化。它将事件路由到 Datadog 和第一方（1P）事件日志系统。

## 核心内容详解

### 主要函数

#### `initializeAnalyticsSink(): void`
初始化分析 Sink。应该在应用启动时调用。
- 幂等设计：多次调用后序为无操作
- 附加 Sink 后，之前排队的事件会被排空

#### `initializeAnalyticsGates(): void`
在启动期间初始化分析门控（Gates）。
- 从服务器更新门控值
- 早期事件使用上次会话的缓存值以避免初始化期间数据丢失

#### `logEventImpl(eventName, metadata): void`
同步事件日志实现。处理流程：
1. 检查事件是否应被采样
2. 如果采样结果为 0，丢弃事件
3. 如果启用 Datadog，发送事件（剥离 `_PROTO_*` 键）
4. 发送到 1P 事件日志（保留完整负载）

#### `logEventAsyncImpl(eventName, metadata): Promise<void>`
异步包装器，当前只是同步实现的 Promise 包装（因为剩余 Sink 都是 fire-and-forget）。

### Sink 路由逻辑

```
事件产生
    ↓
采样检查
    ↓
┌─────────────┬─────────────┐
↓             ↓             ↓
Datadog    1P 事件日志    (已移除 Segment)
(剥离PII)   (完整负载)
```

### 门控配置

- `tengu_log_datadog_events` — Datadog 事件日志开关
- 使用 GrowthBook 动态配置管理
- 支持缓存回退：如果未初始化，使用上次会话的缓存值

## 设计要点

1. **双 Sink 架构** — 同时支持 Datadog 和 1P 事件日志
2. **数据隔离** — Datadog 接收剥离 `_PROTO_*` 的数据，1P 接收完整数据
3. **动态采样** — 支持基于事件名称的采样率配置
4. **缓存策略** — 启动期间使用缓存值，后台更新新值

## 与其他文件的关系

- **调用**: `index.ts` 的 `attachAnalyticsSink()`
- **依赖**: `datadog.ts`, `firstPartyEventLogger.ts`, `growthbook.ts`
- **被调用**: `main.tsx` 的 `setupBackend()`

## 注意事项

- Segment 已被移除，仅剩 Datadog 和 1P 两个 Sink
- `_PROTO_*` 键在发送到 Datadog 前被剥离，保护敏感数据
- 采样配置来自 `tengu_event_sampling_config` 动态配置
- 门控状态缓存于模块级别，支持启动期间的快速检查
