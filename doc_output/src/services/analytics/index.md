# index.ts — 分析服务公共 API

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/analytics/index.ts`
- **所属模块**: Analytics Service
- **功能类型**: 公共 API / 事件路由

## 功能概述

该模块是 Claude CLI 中分析事件的入口点。它提供事件日志的公共 API，负责将事件路由到已连接的 Sink（数据接收器）。设计目标是**无任何依赖**，以避免导入循环。

## 核心内容详解

### 类型定义

#### `AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS`
标记类型，用于强制验证分析元数据不包含敏感数据（代码片段、文件路径等）。通过类型转换使用，确保开发者明确确认数据安全性。

#### `AnalyticsMetadata_I_VERIFIED_THIS_IS_PII_TAGGED`
标记类型，用于标记将路由到 PII 标记的 proto 列的值。这些值目的地有特权访问控制。

### 主要函数

#### `stripProtoFields<V>(metadata): Record<string, V>`
从元数据中剥离 `_PROTO_*` 键。用于：
- 发送到通用访问存储前的清理
- 防止未来未知的 `_PROTO_foo` 静默进入 BQ JSON blob

#### `attachAnalyticsSink(newSink: AnalyticsSink): void`
附加分析 Sink 以接收所有事件。特点：
- 幂等设计：如果 Sink 已附加，则为无操作
- 异步排空队列：使用 `queueMicrotask` 避免阻塞启动路径
- 记录队列大小以帮助调试分析初始化时机

#### `logEvent(eventName, metadata): void`
同步记录事件到分析后端。
- 如果未附加 Sink，事件被排队并在 Sink 附加时排空
- 元数据类型限制为 `boolean | number | undefined`，避免意外记录字符串

#### `logEventAsync(eventName, metadata): Promise<void>`
异步版本的事件记录函数。

### 事件队列机制

```
事件产生 → [事件队列] → Sink 附加 → 异步排空 → Datadog/1P
                ↓
            Sink 未附加时暂存
```

## 设计要点

1. **零依赖设计** — 模块无任何导入，避免导入循环
2. **事件队列** — Sink 初始化前的事件不会丢失，而是排队等待
3. **类型安全** — 使用标记类型强制开发者验证数据安全性
4. **异步排空** — 队列排空异步执行，不阻塞启动路径
5. **幂等性** — `attachAnalyticsSink` 可安全多次调用

## 与其他文件的关系

- **被调用**: 应用各处的 `logEvent()` / `logEventAsync()`
- **实现**: `sink.ts` 实现具体的 `AnalyticsSink` 接口
- **关联**: `datadog.ts`, `firstPartyEventLogger.ts` 作为实际后端

## 注意事项

- 字符串元数据需显式类型转换为 `AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS`
- `_PROTO_*` 键专门用于 PII 标记的数据列，会自动剥离
- 事件可能基于 `tengu_event_sampling_config` 动态配置进行采样
- 测试时使用 `_resetForTesting()` 重置状态
