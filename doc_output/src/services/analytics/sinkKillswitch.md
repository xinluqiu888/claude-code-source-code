# sinkKillswitch.ts — 分析 Sink 紧急停止开关

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/analytics/sinkKillswitch.ts`
- **所属模块**: Analytics Service
- **功能类型**: 紧急停止控制

## 功能概述

该模块提供每个 Sink（数据接收器）级别的分析紧急停止功能。通过 GrowthBook 动态配置控制单个 Sink 的启用/禁用状态。

## 核心内容详解

### 配置名称

```typescript
SINK_KILLSWITCH_CONFIG_NAME = 'tengu_frond_boric'  // 混淆名称
```

### Sink 类型

```typescript
type SinkName = 'datadog' | 'firstParty'
```

支持控制的 Sink：
- `datadog` — Datadog 日志服务
- `firstParty` — 第一方事件日志服务

### 主要函数

#### `isSinkKilled(sink: SinkName): boolean`
检查指定 Sink 是否被紧急停止。

**配置格式：**
```json
{
  "datadog": true,      // true = 停止 Datadog
  "firstParty": false   // false 或未定义 = 保持启用
}
```

**行为：**
- `true` — 停止该 Sink 的所有事件分发
- `false` 或未定义 — Sink 保持启用
- 默认 `{}`（无停止）
- 故障开放：配置缺失或格式错误时 Sink 保持启用

**注意：**
- 必须在事件分发调用点使用
- 不能在 `is1PEventLoggingEnabled()` 内部调用（会导致递归）

## 设计要点

1. **Sink 级控制** — 可独立控制每个 Sink，不影响其他
2. **故障开放** — 配置问题时不中断服务
3. **混淆名称** — 配置键使用不易猜测的名称
4. **GrowthBook 集成** — 通过动态配置实现实时控制

## 与其他文件的关系

- **被调用**: `sink.ts` 的路由逻辑
- **依赖**: `growthbook.ts` 的 `getDynamicConfig_CACHED_MAY_BE_STALE()`
- **注意**: 不能从 `is1PEventLoggingEnabled()` 调用（递归风险）

## 注意事项

- 紧急停止在配置更新后立即生效
- 已排队的失败事件在退避重试期间仍会继续尝试
- 配置变更不需要重启应用
- 故障开放设计确保服务可用性优先于控制精确性
