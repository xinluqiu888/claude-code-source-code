# timeBasedMCConfig.ts — 基于时间的微压缩配置

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/compact/timeBasedMCConfig.ts`
- **作用域**: 基于时间间隔的微压缩配置管理
- **主要导出**:
  - `getTimeBasedMCConfig`: 获取时间微压缩配置
  - `TimeBasedMCConfig`: 配置类型定义

## 功能概述

配置基于时间的微压缩触发器。当距上次主循环助手消息的时间间隔超过阈值时，触发内容清理的微压缩。这是因为服务端 prompt cache 已经过期，完整前缀将被重写，提前清理旧工具结果可以缩小重写的数据量。

## 核心内容详解

### 配置类型

```typescript
export type TimeBasedMCConfig = {
  /** 主开关 */
  enabled: boolean
  /** 触发阈值（分钟）：距上次助手消息的时间差超过此值时触发 */
  gapThresholdMinutes: number
  /** 保留最近 N 个可压缩工具结果 */
  keepRecent: number
}
```

### 默认配置

```typescript
const TIME_BASED_MC_CONFIG_DEFAULTS: TimeBasedMCConfig = {
  enabled: false,           // 默认关闭
  gapThresholdMinutes: 60,  // 60 分钟，服务端 cache 1 小时 TTL
  keepRecent: 5,           // 保留最近 5 个
}
```

### GrowthBook 集成

```typescript
export function getTimeBasedMCConfig(): TimeBasedMCConfig {
  return getFeatureValue_CACHED_MAY_BE_STALE<TimeBasedMCConfig>(
    'tengu_slate_heron',
    TIME_BASED_MC_CONFIG_DEFAULTS,
  )
}
```

## 设计要点

1. **时间触发**: 基于距上次助手消息的时间差
2. **安全阈值**: 60 分钟对应服务端 1 小时 cache TTL
3. **预请求执行**: 在 API 调用前执行，发送缩小后的 prompt
4. **主线程专用**: 子代理生命周期短，不适用基于间隔的清理

## 与其他文件的关系

- **microCompact.ts**: 调用 `getTimeBasedMCConfig` 和 `evaluateTimeBasedTrigger`
- **growthbook.ts**: 提供特性标志读取功能

## 注意事项

1. **GrowthBook Flag**: `tengu_slate_heron`
2. **缓存假设**: 假设超过 60 分钟后服务端 cache 已失效
3. **互斥性**: 时间触发时会跳过缓存微压缩（cache-editing）
