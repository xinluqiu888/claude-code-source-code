# stopTask.ts — 任务停止逻辑

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/stopTask.ts`
- **类型**: TypeScript 模块
- **主要功能**: 提供停止运行中任务的共享逻辑

## 功能概述

本文件提供了停止运行中任务的共享逻辑，被TaskStopTool(LLM调用)和SDK stop_task控制请求使用。支持通过任务ID查找、验证和终止任务，并提供特定任务类型的特殊处理。

## 核心内容详解

### 类型定义

#### `StopTaskError`
停止任务错误类：
```typescript
class StopTaskError extends Error {
  code: 'not_found' | 'not_running' | 'unsupported_type'
}
```

错误码：
- `not_found`: 未找到任务
- `not_running`: 任务未在运行
- `unsupported_type`: 不支持的任务类型

#### `StopTaskContext`
停止任务上下文：
```typescript
type StopTaskContext = {
  getAppState: () => AppState
  setAppState: (f: (prev: AppState) => AppState) => void
}
```

#### `StopTaskResult`
停止任务结果：
```typescript
type StopTaskResult = {
  taskId: string
  taskType: string
  command: string | undefined
}
```

### 主要函数

#### `stopTask`
停止指定ID的任务：

执行流程：
1. 通过getAppState()获取应用状态
2. 查找任务：
   - 如果未找到，抛出StopTaskError('not_found')
3. 检查状态：
   - 如果状态不是'running'，抛出StopTaskError('not_running')
4. 获取任务实现：
   - 通过getTaskByType(task.type)获取Task实现
   - 如果不支持，抛出StopTaskError('unsupported_type')
5. 调用taskImpl.kill()终止任务
6. **Bash任务特殊处理**：
   - 抑制"exit code 137"通知(噪音)
   - 原子性设置notified标志
   - 直接发送task_terminated SDK事件
7. 返回StopTaskResult

Bash任务特殊处理详情：
```typescript
if (isLocalShellTask(task)) {
  // 设置notified标志
  setAppState(prev => {
    if (prevTask.notified) return prev
    suppressed = true
    return { ...prev, tasks: { ...prev.tasks, [taskId]: { ...prevTask, notified: true } } }
  })
  // 发送SDK事件
  if (suppressed) {
    emitTaskTerminatedSdk(taskId, 'stopped', { toolUseId, summary })
  }
}
```

设计原因：
- 抑制XML通知避免"exit code 137"噪音
- 但SDK消费者仍需要看到任务关闭
- 直接emitTaskTerminatedSdk发送SDK事件

## 设计要点

1. **统一接口**：提供统一的stopTask接口，无论调用来源(LLM或SDK)
2. **错误分类**：通过错误码区分不同失败原因，便于调用者处理
3. **Bash特殊处理**：Bash任务抑制通知但发送SDK事件
4. **原子性操作**：notified标志检查+设置是原子性的

## 与其他文件的关系

- **导入**：
  - `AppState` from state/AppState
  - `TaskStateBase` from Task
  - `getTaskByType` from tasks - 获取任务类型实现
  - `emitTaskTerminatedSdk` from sdkEventQueue - SDK事件
  - `isLocalShellTask` from LocalShellTask/guards - Bash任务检测

- **被调用者**：
  - TaskStopTool - LLM调用的停止任务工具
  - SDK stop_task控制请求处理

## 注意事项

1. **错误码检查**：调用者可通过error.code区分失败原因
2. **Bash通知抑制**：137退出码(被信号终止)的通知被抑制
3. **SDK事件**：即使抑制XML通知，SDK消费者仍会收到task_terminated事件
4. **命令返回**：返回的command对Bash任务是实际命令，其他任务是description
