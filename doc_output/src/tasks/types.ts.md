# types.ts — 任务状态联合类型

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/types.ts`
- **类型**: TypeScript 类型定义文件
- **主要功能**: 定义所有具体任务状态类型的联合类型

## 功能概述

本文件定义了TaskState联合类型，包含所有具体的任务状态类型。用于需要处理任意任务类型的组件。同时定义了BackgroundTaskState和isBackgroundTask函数，用于后台任务指示器的筛选逻辑。

## 核心内容详解

### 类型导入

导入所有具体任务状态类型：
- `DreamTaskState` from DreamTask/DreamTask
- `InProcessTeammateTaskState` from InProcessTeammateTask/types
- `LocalAgentTaskState` from LocalAgentTask/LocalAgentTask
- `LocalShellTaskState` from LocalShellTask/guards
- `LocalWorkflowTaskState` from LocalWorkflowTask/LocalWorkflowTask
- `MonitorMcpTaskState` from MonitorMcpTask/MonitorMcpTask
- `RemoteAgentTaskState` from RemoteAgentTask/RemoteAgentTask

### 类型定义

#### `TaskState`
所有具体任务状态类型的联合：
```typescript
type TaskState =
  | LocalShellTaskState
  | LocalAgentTaskState
  | RemoteAgentTaskState
  | InProcessTeammateTaskState
  | LocalWorkflowTaskState
  | MonitorMcpTaskState
  | DreamTaskState
```

#### `BackgroundTaskState`
可出现在后台任务指示器中的任务类型：
```typescript
type BackgroundTaskState =
  | LocalShellTaskState
  | LocalAgentTaskState
  | RemoteAgentTaskState
  | InProcessTeammateTaskState
  | LocalWorkflowTaskState
  | MonitorMcpTaskState
  | DreamTaskState
```

注意：与TaskState完全相同，但语义上区分"任何任务"和"后台任务"。

### 类型守卫

#### `isBackgroundTask`
检查任务是否应显示在后台任务指示器中：

条件：
1. 状态为'running'或'pending'
2. 非显式前台运行(isBackgrounded !== false)

实现逻辑：
```typescript
function isBackgroundTask(task: TaskState): task is BackgroundTaskState {
  // 1. 非运行/挂起状态 → 不是后台任务
  if (task.status !== 'running' && task.status !== 'pending') {
    return false
  }
  // 2. 显式前台任务(isBackgrounded === false) → 不是后台任务
  if ('isBackgrounded' in task && task.isBackgrounded === false) {
    return false
  }
  return true
}
```

## 设计要点

1. **联合类型**：将分散在各子目录的任务状态统一为联合类型
2. **类型安全**：组件使用TaskState可以处理任意任务类型
3. **后台筛选**：isBackgroundTask统一后台任务判断逻辑
4. **语义区分**：TaskState和BackgroundTaskState虽然相同，但语义上区分用途

## 与其他文件的关系

- **导入**：
  - 所有任务子目录的类型定义

- **被导入**：
  - 需要处理任意任务类型的组件
  - 后台任务指示器组件
  - 任务管理工具函数

## 注意事项

1. **同步更新**：添加新任务类型时需要同时更新TaskState和BackgroundTaskState
2. **后台判断**：isBackgrounded === false表示显式前台运行，不显示在后台指示器
3. **状态筛选**：只有running/pending状态才可能成为后台任务
