# teammateViewHelpers.ts — 队友视图状态管理助手

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/state/teammateViewHelpers.ts`
- **类型**: TypeScript 模块
- **导出内容**: `enterTeammateView()` 函数、`exitTeammateView()` 函数、`stopOrDismissAgent()` 函数
- **依赖关系**:
  - 导入: `services/analytics/index.js`, `Task.js`, `tasks/LocalAgentTask/LocalAgentTask.js`
  - 常量: `PANEL_GRACE_MS` (内联自 framework.ts)

## 功能概述

本文件提供用于管理队友（子代理/子任务）视图状态的辅助函数。它处理进入/退出队友转录视图的转换逻辑，包括状态保留、消息清理、驱逐计划等。这些函数封装了视图切换的复杂状态管理，确保资源正确释放和 UI 状态同步。

## 核心内容详解

### 1. PANEL_GRACE_MS 常量 (第6-7行)

```typescript
const PANEL_GRACE_MS = 30_000  // 30秒宽限期
```

从 framework.ts 内联的常量，表示面板任务完成后保留的宽限时间。避免循环导入。

### 2. isLocalAgent() 函数 (第14-21行)

本地代理任务的类型守卫，内联实现以避免循环依赖。

```typescript
function isLocalAgent(task: unknown): task is LocalAgentTaskState {
  return (
    typeof task === 'object' &&
    task !== null &&
    'type' in task &&
    task.type === 'local_agent'
  )
}
```

### 3. release() 函数 (第28-38行)

将任务释放回存根形式的内部函数。

**功能**:
- 设置 `retain: false`（允许驱逐）
- 清除 `messages`（释放消息内存）
- 设置 `diskLoaded: false`
- 如果任务处于终止状态，设置 `evictAfter` 时间戳

**实现**:
```typescript
function release(task: LocalAgentTaskState): LocalAgentTaskState {
  return {
    ...task,
    retain: false,
    messages: undefined,
    diskLoaded: false,
    evictAfter: isTerminalTaskStatus(task.status)
      ? Date.now() + PANEL_GRACE_MS
      : undefined,
  }
}
```

### 4. enterTeammateView() 函数 (第46-81行)

进入队友转录视图的入口函数。

**参数**:
- `taskId: string`: 要查看的队友任务ID
- `setAppState`: 状态更新函数

**功能**:
1. 记录分析事件 `tengu_transcript_view_enter`
2. 设置 `viewingAgentTaskId` 为指定任务
3. 设置 `viewSelectionMode` 为 `'viewing-agent'`
4. 如果切换到另一个代理，释放之前的代理
5. 对于本地代理任务，设置 `retain: true` 和 `evictAfter: undefined`

**状态转换逻辑**:
```
切换条件 (switching):
- 之前有其他 viewId (prevId !== undefined)
- 且不是同一个任务 (prevId !== taskId)
- 且之前的任务是本地代理 (isLocalAgent(prevTask))
- 且之前被保留 (prevTask.retain)

保留条件 (needsRetain):
- 是本地代理 (isLocalAgent(task))
- 且未被保留或已设置驱逐时间 (!task.retain || task.evictAfter !== undefined)

视图需求 (needsView):
- 视图任务ID变化或选择模式不是 viewing-agent
```

### 5. exitTeammateView() 函数 (第88-109行)

退出队友视图并返回主代理视图。

**参数**:
- `setAppState`: 状态更新函数

**功能**:
1. 记录分析事件 `tengu_transcript_view_exit`
2. 清除 `viewingAgentTaskId`（设为 `undefined`）
3. 设置 `viewSelectionMode` 为 `'none'`
4. 如果正在查看的代理被保留，释放它
5. 对于终止状态的任务，设置 `evictAfter` 以便行短暂停留

**返回状态**:
```typescript
{
  ...prev,
  viewingAgentTaskId: undefined,
  viewSelectionMode: 'none',
  // 如果任务被保留，还更新 tasks 中对应任务
}
```

### 6. stopOrDismissAgent() 函数 (第116-141行)

上下文敏感的停止或关闭代理操作。

**行为**:
- 运行中 (`running`): 中止任务（调用 `abortController.abort()`）
- 终止状态 (`completed`/`failed`/`killed`): 关闭（设置 `evictAfter: 0`）

**参数**:
- `taskId: string`: 目标任务ID
- `setAppState`: 状态更新函数

**特殊处理**:
- 如果正在查看被关闭的代理，同时退出视图
- 使用 `evictAfter: 0` 使过滤器立即隐藏

**状态更新**:
```typescript
return {
  ...prev,
  tasks: {
    ...prev.tasks,
    [taskId]: { ...release(task), evictAfter: 0 },
  },
  ...(viewingThis && {
    viewingAgentTaskId: undefined,
    viewSelectionMode: 'none',
  }),
}
```

## 设计要点

1. **避免循环依赖**: 使用内联类型守卫和内联常量
2. **函数式更新**: 所有状态更新使用展开操作符创建新对象
3. **条件优化**: 只在必要时才更新 `tasks`（切换或需要保留时）
4. **分析埋点**: 进入/退出视图都有对应的事件记录
5. **资源管理**: 正确管理 `retain`、`evictAfter` 和消息生命周期

## 与其他文件的关系

- **AppStateStore.ts**: 操作 AppState 中的任务和视图状态
- **Task.ts**: 使用 `isTerminalTaskStatus` 判断任务状态
- **LocalAgentTask**: 操作 LocalAgentTaskState
- **framework.ts**: PANEL_GRACE_MS 的原始定义位置

## 使用场景

```typescript
// 进入队友视图
enterTeammateView(agentTaskId, setAppState)

// 退出队友视图返回主视图
exitTeammateView(setAppState)

// 停止运行中的代理或关闭已完成的代理
stopOrDismissAgent(agentTaskId, setAppState)
```

## 注意事项

1. **循环依赖**: 注意此文件与 LocalAgentTask 之间的导入关系
2. **状态一致性**: 确保 `viewingAgentTaskId` 和 `viewSelectionMode` 同步更新
3. **任务保留**: `retain` 标志控制任务是否保留在内存中
4. **驱逐逻辑**: `evictAfter` 为 0 表示立即驱逐，为时间戳表示延迟驱逐
