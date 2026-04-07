# autoCompact.ts — 自动压缩逻辑

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/compact/autoCompact.ts`
- **作用域**: 自动压缩触发、阈值计算、失败重试
- **主要导出**:
  - `autoCompactIfNeeded`: 按需执行自动压缩
  - `shouldAutoCompact`: 检查是否应该自动压缩
  - `getAutoCompactThreshold`: 获取自动压缩阈值
  - `calculateTokenWarningState`: 计算 token 警告状态
  - `isAutoCompactEnabled`: 检查自动压缩是否启用

## 功能概述

实现对话上下文的自动压缩功能。当 token 使用量超过配置的阈值时，自动触发压缩以释放上下文空间，避免达到模型的硬性限制。

## 核心内容详解

### 阈值常量

```typescript
export const AUTOCOMPACT_BUFFER_TOKENS = 13_000
export const WARNING_THRESHOLD_BUFFER_TOKENS = 20_000
export const ERROR_THRESHOLD_BUFFER_TOKENS = 20_000
export const MANUAL_COMPACT_BUFFER_TOKENS = 3_000
const MAX_CONSECUTIVE_AUTOCOMPACT_FAILURES = 3  // 熔断器阈值
```

### 主要函数

#### `getAutoCompactThreshold(model)`
计算自动压缩阈值：
```typescript
// 有效上下文窗口 = 模型窗口 - 预留输出 tokens
// 自动压缩阈值 = 有效窗口 - 13,000 buffer
```

支持通过 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 环境变量按百分比覆盖。

#### `calculateTokenWarningState(tokenUsage, model)`
计算当前 token 使用状态的警告级别：
- `percentLeft`: 剩余百分比
- `isAboveWarningThreshold`: 超过警告阈值
- `isAboveErrorThreshold`: 超过错误阈值
- `isAboveAutoCompactThreshold`: 超过自动压缩阈值
- `isAtBlockingLimit`: 达到阻塞限制（无法继续）

#### `shouldAutoCompact(messages, model, querySource, snipTokensFreed)`
判断是否应该执行自动压缩：

1. **递归保护**: 避免在 `session_memory`、`compact`、`marble_origami` 等源中触发
2. **功能开关检查**: 
   - `DISABLE_COMPACT`
   - `DISABLE_AUTO_COMPACT`
   - 用户配置 `autoCompactEnabled`
3. **特性标志**:
   - `REACTIVE_COMPACT`: 反应式压缩模式
   - `CONTEXT_COLLAPSE`: 上下文折叠模式
4. **Token 阈值检查**: 比较当前使用量与自动压缩阈值

#### `autoCompactIfNeeded(messages, toolUseContext, cacheSafeParams, querySource, tracking, snipTokensFreed)`
执行自动压缩的主要逻辑：

1. **熔断器检查**: 连续失败超过 3 次后停止尝试
2. **条件检查**: 调用 `shouldAutoCompact`
3. **优先尝试会话记忆压缩**: 调用 `trySessionMemoryCompaction`
4. **回退到传统压缩**: 调用 `compactConversation`
5. **状态更新**: 成功后重置失败计数，失败后递增

### 追踪状态

```typescript
export type AutoCompactTrackingState = {
  compacted: boolean        // 本轮是否已压缩
  turnCounter: number       // 轮次计数器
  turnId: string           // 轮次唯一 ID
  consecutiveFailures?: number  // 连续失败次数
}
```

## 设计要点

1. **熔断器机制**: 防止在无法恢复的情况下无限重试
2. **会话记忆优先**: 优先使用轻量级的会话记忆压缩
3. **多重保护**: 递归保护、功能开关、特性标志多层防护
4. **详细遥测**: 记录压缩事件和失败原因

## 与其他文件的关系

- **compact.ts**: 传统压缩实现
- **sessionMemoryCompact.ts**: 会话记忆压缩实现
- **postCompactCleanup.ts**: 压缩后清理
- **growthbook.ts**: 特性标志检查

## 注意事项

1. **环境变量**:
   - `CLAUDE_CODE_AUTO_COMPACT_WINDOW`: 覆盖自动压缩窗口大小
   - `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`: 按百分比设置阈值
   - `CLAUDE_CODE_BLOCKING_LIMIT_OVERRIDE`: 覆盖阻塞限制

2. **querySource 过滤**:
   - `session_memory`: 避免死锁
   - `compact`: 避免递归
   - `marble_origami`: 避免破坏主线程状态
