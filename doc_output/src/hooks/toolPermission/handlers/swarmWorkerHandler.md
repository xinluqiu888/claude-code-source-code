# swarmWorkerHandler.ts — Swarm Worker 权限处理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/toolPermission/handlers/swarmWorkerHandler.ts`
- **类型**: TypeScript 模块
- **导出函数**: `handleSwarmWorkerPermission`
- **导出类型**: `SwarmWorkerPermissionParams`

## 功能概述

本模块处理 Swarm Worker（队友/子代理）的权限流程：
1. 尝试分类器自动审批（仅 bash 命令）
2. 通过 Mailbox 将权限请求转发给 Leader
3. 注册回调等待 Leader 响应
4. 设置等待指示器
5. 支持中止信号处理

## 核心内容详解

### 参数类型

```typescript
type SwarmWorkerPermissionParams = {
  ctx: PermissionContext                    // 权限上下文
  description: string                        // 工具描述
  pendingClassifierCheck?: PendingClassifierCheck | undefined  // 待处理的分类器检查
  updatedInput: Record<string, unknown> | undefined  // 更新的输入
  suggestions: PermissionUpdate[] | undefined  // 权限建议
}
```

### 早期返回条件

```typescript
if (!isAgentSwarmsEnabled() || !isSwarmWorker()) {
  return null
}
```

如果 Swarm 功能未启用或当前不是 Swarm Worker，直接返回 null 让调用者回退到交互式处理。

### 处理流程

**1. 分类器自动审批**
```typescript
const classifierResult = feature('BASH_CLASSIFIER')
  ? await ctx.tryClassifier?.(params.pendingClassifierCheck, updatedInput)
  : null
if (classifierResult) {
  return classifierResult
}
```

与协调器类似，先尝试分类器自动批准。

**2. 转发给 Leader**

创建权限请求并通过 Mailbox 发送：
```typescript
const request = createPermissionRequest({
  toolName: ctx.tool.name,
  toolUseId: ctx.toolUseID,
  input: ctx.input,
  description,
  permissionSuggestions: suggestions,
})
```

**3. 注册回调**

在发送请求前注册回调，避免竞态：
```typescript
registerPermissionCallback({
  requestId: request.id,
  toolUseId: ctx.toolUseID,
  onAllow: async (allowedInput, permissionUpdates, feedback, contentBlocks) => {
    // 处理 Leader 允许响应
  },
  onReject: (feedback, contentBlocks) => {
    // 处理 Leader 拒绝响应
  },
})
```

**4. 设置等待指示器**

```typescript
ctx.toolUseContext.setAppState(prev => ({
  ...prev,
  pendingWorkerRequest: {
    toolName: ctx.tool.name,
    toolUseId: ctx.toolUseID,
    description,
  },
}))
```

**5. 中止处理**

```typescript
ctx.toolUseContext.abortController.signal.addEventListener(
  'abort',
  () => {
    if (!claim()) return
    clearPendingRequest()
    ctx.logCancelled()
    resolveOnce(ctx.cancelAndAbort(undefined, true))
  },
  { once: true },
)
```

### 错误回退

如果 Swarm 权限提交失败：
```typescript
try {
  // ... 上述流程
} catch (error) {
  logError(toError(error))
  return null  // 回退到本地 UI 处理
}
```

## 设计要点

### 1. 转发模式

Worker 不直接做决策，而是将请求转发给 Leader，由 Leader 统一决策。

### 2. 回调先注册

必须先注册回调再发送请求，避免 Leader 响应比回调注册还快。

### 3. 状态指示

设置 `pendingWorkerRequest` 状态，让 UI 显示"等待 Leader 批准"指示器。

### 4. 原子性保证

使用 `createResolveOnce` 的 `claim()` 确保只有一个响应者能最终解决。

### 5. 清理回调

无论结果如何，都要调用 `clearPendingRequest()` 清除等待指示器。

## 与其他文件的关系

- **PermissionContext.ts**: 使用 `PermissionContext` 和 `createResolveOnce`
- **agentSwarmsEnabled.ts**: 使用 `isAgentSwarmsEnabled`, `isSwarmWorker`
- **permissionSync.ts**: 使用 `createPermissionRequest`, `sendPermissionRequestViaMailbox`
- **useSwarmPermissionPoller.ts**: 使用 `registerPermissionCallback`

## 注意事项

1. **特性依赖**: 依赖 Swarm 功能和 BASH_CLASSIFIER 特性标志
2. **Mailbox 通信**: 所有通信通过 Mailbox 完成，Leader 和 Worker 可能不在同一进程
3. **输入合并**: Leader 返回的 `allowedInput` 需要与原始输入合并
4. **中止传播**: Worker 中止时需要通知 Leader，但实际中止由 Worker 执行
5. **错误回退**: Mailbox 失败时回退到本地交互式处理
