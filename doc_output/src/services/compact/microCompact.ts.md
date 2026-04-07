# microCompact.ts — 微压缩实现

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/compact/microCompact.ts`
- **作用域**: 轻量级消息压缩、缓存编辑、工具结果清理
- **主要导出**:
  - `microcompactMessages`: 执行微压缩
  - `estimateMessageTokens`: 估算消息 token 数
  - `evaluateTimeBasedTrigger`: 评估时间触发器
  - `PendingCacheEdits`: 待处理缓存编辑类型
  - `MicrocompactResult`: 微压缩结果类型

## 功能概述

实现轻量级的微压缩功能，在发送 API 请求前清理工具结果以节省 tokens。支持多种压缩路径：基于时间的压缩、缓存微压缩（cache-editing）和传统微压缩。

## 核心内容详解

### 可压缩工具列表

```typescript
const COMPACTABLE_TOOLS = new Set<string>([
  FILE_READ_TOOL_NAME,
  ...SHELL_TOOL_NAMES,
  GREP_TOOL_NAME,
  GLOB_TOOL_NAME,
  WEB_SEARCH_TOOL_NAME,
  WEB_FETCH_TOOL_NAME,
  FILE_EDIT_TOOL_NAME,
  FILE_WRITE_TOOL_NAME,
])
```

### 核心函数

#### `microcompactMessages(messages, toolUseContext?, querySource?)`
主入口函数，按优先级执行：

1. **清除抑制**: `clearCompactWarningSuppression()`
2. **时间触发**: 如果时间间隔超过阈值，执行基于时间的微压缩
3. **缓存微压缩** (ant-only, `CACHED_MICROCOMPACT`):
   - 检查模型支持
   - 注册工具结果
   - 生成缓存编辑块
4. **传统微压缩**: 直接修改消息内容（已移除，始终使用缓存路径或 autocompact）

#### `cachedMicrocompactPath(messages, querySource)`
缓存微压缩路径：

1. 收集可压缩工具 ID
2. 注册工具结果（按用户消息分组）
3. 获取要删除的工具列表
4. 创建 `cache_edits` 块
5. 记录遥测事件
6. 返回带有 `pendingCacheEdits` 的结果

#### `evaluateTimeBasedTrigger(messages, querySource)`
评估时间触发器：

```typescript
export function evaluateTimeBasedTrigger(
  messages: Message[],
  querySource: QuerySource | undefined,
): { gapMinutes: number; config: TimeBasedMCConfig } | null
```

检查条件：
- 功能启用
- 主线程源
- 距上次助手消息时间超过阈值

#### `maybeTimeBasedMicrocompact(messages, querySource)`
执行基于时间的微压缩：

1. 评估触发条件
2. 收集可压缩工具 ID
3. 保留最近 N 个，清除其余
4. 替换工具结果为 `[Old tool result content cleared]`
5. 记录遥测事件
6. 重置缓存微压缩状态

#### `estimateMessageTokens(messages)`
估算消息的 token 数量：

- 文本块：直接估算
- 工具结果：递归计算内容
- 图片/文档：固定 2000 tokens
- Thinking 块：仅计算 thinking 文本
- 其他块：JSON 序列化后估算
- 最终乘以 4/3 作为保守估计

### 缓存微压缩状态管理

```typescript
let cachedMCModule: typeof import('./cachedMicrocompact.js') | null = null
let cachedMCState: import('./cachedMicrocompact.js').CachedMCState | null = null
let pendingCacheEdits: import('./cachedMicrocompact.js').CacheEditsBlock | null = null

export function consumePendingCacheEdits(): CacheEditsBlock | null
export function getPinnedCacheEdits(): PinnedCacheEdits[]
export function pinCacheEdits(userMessageIndex: number, block: CacheEditsBlock): void
export function markToolsSentToAPIState(): void
export function resetMicrocompactState(): void
```

## 设计要点

1. **多级压缩**: 时间触发 > 缓存编辑 > 无压缩
2. **主线程保护**: 缓存微压缩仅对主线程执行
3. **延迟加载**: 缓存微压缩模块延迟加载以支持死代码消除
4. **状态管理**: 维护待处理和已固定的缓存编辑状态

## 与其他文件的关系

- **cachedMicrocompact.ts**: 缓存微压缩核心逻辑
- **timeBasedMCConfig.ts**: 时间触发配置
- **compactWarningState.ts**: 警告状态管理
- **promptCacheBreakDetection.ts**: 缓存断裂检测

## 注意事项

1. **主线程检测**: `isMainThreadSource` 使用 `startsWith` 匹配以支持输出样式变体
2. **缓存假设**: 缓存微压缩假设服务端缓存是热的
3. **冷缓存回退**: 时间触发时跳过缓存编辑，直接内容清除
