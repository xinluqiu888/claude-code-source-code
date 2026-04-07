# hooks.ts — Hook 系统类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/types/hooks.ts`
- **类型**: TypeScript 模块
- **导出内容**: `HookCallback` 类型、`HookResult` 类型、`PermissionRequestResult` 类型、`AggregatedHookResult` 类型，以及各种 Zod schema
- **依赖关系**:
  - 导入: `zod/v4`, `lazySchema.js`, `agentSdkTypes.js`, `message.js`, `PermissionResult.js`, `PermissionRule.js`, `PermissionUpdateSchema.js`, `AppState.js`, `commitAttribution.js`

## 功能概述

本文件定义了 Claude Code Hook 系统的完整类型体系，包括同步和异步 hook 的响应 schema、回调 hook 的类型定义、权限请求结果类型等。它是连接 SDK 类型定义和内部实现的关键桥梁，提供了完整的类型安全和运行时验证。

## 核心内容详解

### 1. isHookEvent() 函数 (第22-24行)

Hook 事件名称的类型守卫：

```typescript
export function isHookEvent(value: string): value is HookEvent {
  return HOOK_EVENTS.includes(value as HookEvent)
}
```

### 2. PromptRequest & PromptResponse (第27-47行)

提示请求/响应协议类型，用于用户交互式提示：

**PromptRequest**:
```typescript
export type PromptRequest = {
  prompt: string      // 请求ID
  message: string     // 显示消息
  options: Array<{    // 选项列表
    key: string
    label: string
    description?: string
  }>
}
```

**PromptResponse**:
```typescript
export type PromptResponse = {
  prompt_response: string  // 请求ID
  selected: string         // 选中的选项key
}
```

### 3. syncHookResponseSchema (第50-166行)

同步 Hook 响应的 Zod Schema，包含丰富的字段：

**基础字段**:
- `continue`: 是否继续执行（默认 true）
- `suppressOutput`: 是否隐藏 stdout（默认 false）
- `stopReason`: 停止原因消息
- `decision`: 决策（'approve' | 'block'）
- `reason`: 决策解释
- `systemMessage`: 系统警告消息

**事件特定输出** (hookSpecificOutput):
根据 `hookEventName` 的不同，包含不同字段：

| Hook 事件 | 特定字段 |
|-----------|----------|
| PreToolUse | permissionDecision, permissionDecisionReason, updatedInput, additionalContext |
| UserPromptSubmit | additionalContext |
| SessionStart | additionalContext, initialUserMessage, watchPaths |
| Setup | additionalContext |
| SubagentStart | additionalContext |
| PostToolUse | additionalContext, updatedMCPToolOutput |
| PostToolUseFailure | additionalContext |
| PermissionDenied | retry |
| Notification | additionalContext |
| PermissionRequest | decision (allow/deny 详情) |
| Elicitation/ElicitationResult | action, content |
| CwdChanged/FileChanged | watchPaths |
| WorktreeCreate | worktreePath |

### 4. hookJSONOutputSchema (第169-176行)

完整的 Hook JSON 输出 Schema，支持同步和异步：

```typescript
export const hookJSONOutputSchema = lazySchema(() => {
  const asyncHookResponseSchema = z.object({
    async: z.literal(true),
    asyncTimeout: z.number().optional(),
  })
  return z.union([asyncHookResponseSchema, syncHookResponseSchema()])
})
```

### 5. 类型守卫函数 (第182-193行)

**isSyncHookJSONOutput()**:
```typescript
export function isSyncHookJSONOutput(json: HookJSONOutput): json is SyncHookJSONOutput {
  return !('async' in json && json.async === true)
}
```

**isAsyncHookJSONOutput()**:
```typescript
export function isAsyncHookJSONOutput(json: HookJSONOutput): json is AsyncHookJSONOutput {
  return 'async' in json && json.async === true
}
```

### 6. 类型兼容性断言 (第196-200行)

编译时断言确保 SDK 类型和 Zod 类型匹配：

```typescript
type _assertSDKTypesMatch = Assert<IsEqual<SchemaHookJSONOutput, HookJSONOutput>>
```

### 7. HookCallbackContext (第203-208行)

传递给回调 hook 的上下文：

```typescript
export type HookCallbackContext = {
  getAppState: () => AppState
  updateAttributionState: (updater: (prev: AttributionState) => AttributionState) => void
}
```

### 8. HookCallback 类型 (第211-226行)

回调类型的 Hook 定义：

```typescript
export type HookCallback = {
  type: 'callback'
  callback: (
    input: HookInput,
    toolUseID: string | null,
    abort: AbortSignal | undefined,
    hookIndex?: number,           // 用于 SessionStart hooks
    context?: HookCallbackContext, // 可选状态访问
  ) => Promise<HookJSONOutput>
  timeout?: number       // 超时秒数
  internal?: boolean     // 是否内部 hook（排除在指标外）
}
```

### 9. HookCallbackMatcher (第228-232行)

Hook 匹配器配置：

```typescript
export type HookCallbackMatcher = {
  matcher?: string       // 匹配模式（可选）
  hooks: HookCallback[]  // Hook 列表
  pluginName?: string    // 插件名称
}
```

### 10. HookProgress (第234-241行)

Hook 执行进度信息：

```typescript
export type HookProgress = {
  type: 'hook_progress'
  hookEvent: HookEvent
  hookName: string
  command: string
  promptText?: string
  statusMessage?: string
}
```

### 11. PermissionRequestResult (第248-259行)

权限请求结果类型：

```typescript
export type PermissionRequestResult =
  | { behavior: 'allow'; updatedInput?: Record<string, unknown>; updatedPermissions?: PermissionUpdate[] }
  | { behavior: 'deny'; message?: string; interrupt?: boolean }
```

### 12. HookResult (第260-275行)

单个 Hook 执行结果：

| 字段 | 说明 |
|------|------|
| `message` | 返回的消息 |
| `systemMessage` | 系统消息 |
| `blockingError` | 阻塞错误 |
| `outcome` | 结果类型（success/blocking/non_blocking_error/cancelled） |
| `preventContinuation` | 是否阻止继续 |
| `stopReason` | 停止原因 |
| `permissionBehavior` | 权限行为 |
| `additionalContext` | 额外上下文 |
| `updatedInput` | 更新的输入 |
| `permissionRequestResult` | 权限请求结果 |
| `retry` | 是否重试 |

### 13. AggregatedHookResult (第277-290行)

多个 Hook 的聚合结果：

```typescript
export type AggregatedHookResult = {
  message?: Message
  blockingErrors?: HookBlockingError[]
  preventContinuation?: boolean
  stopReason?: string
  hookPermissionDecisionReason?: string
  permissionBehavior?: PermissionResult['behavior']
  additionalContexts?: string[]
  initialUserMessage?: string
  updatedInput?: Record<string, unknown>
  updatedMCPToolOutput?: unknown
  permissionRequestResult?: PermissionRequestResult
  retry?: boolean
}
```

## 设计要点

1. **懒加载 Schema**: 使用 `lazySchema` 避免 Zod 初始化时的循环依赖
2. **联合类型**: 同步/异步响应使用 discriminated union 区分
3. **事件特定输出**: 每个 hook 事件可以有特定的输出字段
4. **类型守卫**: 提供运行时类型检查函数
5. **编译时断言**: 确保内部类型与 SDK 类型兼容

## 与其他文件的关系

- **agentSdkTypes.ts**: 导入基础 Hook 类型定义
- **PermissionRule.js/PermissionUpdateSchema.js**: 权限相关 schema
- **utils/settings/types.ts**: 使用这些类型定义 hook 配置
- **utils/hooks/**: 实际 hook 执行逻辑

## 注意事项

1. **Schema 懒加载**: Zod schema 使用懒加载避免循环导入问题
2. **类型同步**: 修改 schema 时需要确保 SDK 类型同步更新
3. **内部 Hook**: `internal: true` 的 hook 不从指标中排除
4. **超时单位**: `timeout` 字段以秒为单位
