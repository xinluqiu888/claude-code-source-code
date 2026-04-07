# selectors.ts — AppState 状态选择器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/state/selectors.ts`
- **类型**: TypeScript 模块
- **导出内容**: `getViewedTeammateTask()` 函数、`getActiveAgentForInput()` 函数、`ActiveAgentForInput` 类型
- **依赖关系**:
  - 导入: `InProcessTeammateTask/types.js`, `LocalAgentTask/LocalAgentTask.js`, `AppStateStore.js`

## 功能概述

本文件包含纯函数选择器，用于从 AppState 中派生计算状态。选择器的设计原则是保持纯粹和简单——仅进行数据提取，不产生副作用。这些选择器主要用于确定用户输入应该路由到哪个代理（主代理或子代理）。

## 核心内容详解

### 1. getViewedTeammateTask() 函数 (第18-40行)

获取当前正在查看的队友任务（如果存在）。

**参数**:
- `appState`: 包含 `viewingAgentTaskId` 和 `tasks` 的 AppState 子集

**返回值**:
- `InProcessTeammateTaskState | undefined`: 返回队友任务状态或未定义

**返回 undefined 的情况**:
1. 没有查看任何队友（`viewingAgentTaskId` 未定义）
2. 任务ID在任务列表中不存在
3. 任务不是进程内队友任务类型

**实现逻辑**:
```typescript
export function getViewedTeammateTask(
  appState: Pick<AppState, 'viewingAgentTaskId' | 'tasks'>,
): InProcessTeammateTaskState | undefined {
  const { viewingAgentTaskId, tasks } = appState
  if (!viewingAgentTaskId) return undefined
  const task = tasks[viewingAgentTaskId]
  if (!task) return undefined
  if (!isInProcessTeammateTask(task)) return undefined
  return task
}
```

### 2. ActiveAgentForInput 类型 (第46-50行)

定义输入路由目标的类型安全判别联合：

```typescript
export type ActiveAgentForInput =
  | { type: 'leader' }           // 输入发送到主代理
  | { type: 'viewed'; task: InProcessTeammateTaskState }  // 输入发送到查看的代理
  | { type: 'named_agent'; task: LocalAgentTaskState }    // 输入发送到命名代理
```

### 3. getActiveAgentForInput() 函数 (第59-76行)

确定用户输入应该路由到哪里。

**返回值说明**:
- `{ type: 'leader' }`: 没有查看队友时，输入进入主代理
- `{ type: 'viewed', task }`: 查看代理时，输入进入该代理
- `{ type: 'named_agent', task }`: 查看本地代理时，输入进入命名代理

**使用场景**:
- 输入路由逻辑使用此函数将用户消息定向到正确的代理
- 用于决定消息应该附加到哪个对话上下文

**实现逻辑**:
```typescript
export function getActiveAgentForInput(
  appState: AppState,
): ActiveAgentForInput {
  const viewedTask = getViewedTeammateTask(appState)
  if (viewedTask) {
    return { type: 'viewed', task: viewedTask }
  }

  const { viewingAgentTaskId, tasks } = appState
  if (viewingAgentTaskId) {
    const task = tasks[viewingAgentTaskId]
    if (task?.type === 'local_agent') {
      return { type: 'named_agent', task }
    }
  }

  return { type: 'leader' }
}
```

## 设计要点

1. **纯函数**: 选择器是纯函数，没有副作用，只依赖输入参数
2. **类型安全**: 使用 TypeScript 的判别联合类型确保类型安全
3. **最小依赖**: 每个选择器只依赖 AppState 的必要部分（使用 `Pick`）
4. **可组合性**: `getActiveAgentForInput` 复用 `getViewedTeammateTask`
5. **防御性编程**: 多层检查确保在无效状态下优雅返回

## 与其他文件的关系

- **AppStateStore.ts**: 定义 AppState 类型结构
- **InProcessTeammateTask/types.ts**: 提供队友任务类型和类型守卫
- **LocalAgentTask/LocalAgentTask.js**: 提供本地代理任务类型
- **使用方**: 输入处理系统、消息路由逻辑

## 注意事项

1. 选择器应该保持纯粹，不要在这里添加副作用
2. 如果需要更复杂的状态派生，考虑使用类似 reselect 的库
3. 选择器的性能对高频操作很重要，保持简单高效
