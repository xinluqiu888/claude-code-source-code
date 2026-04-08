# InProcessTeammateTask.tsx — 进程内队友生命周期管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/InProcessTeammateTask/InProcessTeammateTask.tsx`
- **类型**: TSX 模块
- **主要功能**: 实现InProcessTeammate的Task接口，管理队友生命周期

## 功能概述

本组件实现了Task接口用于进程内队友。与LocalAgentTask(后台代理)不同，进程内 teammates:
1. 在同Node.js进程中运行，使用AsyncLocalStorage隔离
2. 具有团队感知身份(agentName@teamName)
3. 支持计划模式审批流程
4. 可以是空闲(等待工作)或活跃(处理中)状态

## 核心内容详解

### InProcessTeammateTask 对象

实现Task接口的核心对象：
```typescript
export const InProcessTeammateTask: Task = {
  name: 'InProcessTeammateTask',
  type: 'in_process_teammate',
  async kill(taskId, setAppState) {
    killInProcessTeammate(taskId, setAppState)
  }
}
```

### 主要函数

#### `requestTeammateShutdown`
请求队友关闭：
- 检查任务是否正在运行且未已请求关闭
- 设置`shutdownRequested`标志
- 用于优雅地请求队友在合适时机停止

#### `appendTeammateMessage`
追加消息到队友对话历史：
- 用于缩放视图显示队友对话
- 仅在任务运行时可追加
- 使用`appendCappedMessage`限制消息数量

#### `injectUserMessageToTeammate`
注入用户消息到队友待处理队列：
- 用于查看队友转录时向其发送消息
- 同时添加到task.messages以便立即显示在转录中
- 仅在非终止状态下允许注入
- 创建用户消息并追加到messages数组

#### `findTeammateTaskByAgentId`
通过agent ID查找队友任务：
- 从AppState中查找指定agentId的任务
- 优先返回运行中的任务(处理重名旧任务)
- 如果没有运行中的任务，返回第一个匹配作为fallback

#### `getAllInProcessTeammateTasks`
获取所有进程内队友任务：
- 从AppState.tasks中过滤出所有in_process_teammate类型任务
- 返回数组形式

#### `getRunningTeammatesSorted`
获取按字母排序的运行中队友：
- 过滤出运行中状态的队友
- 按agentName字母排序
- 被TeammateSpinnerTree、PromptInput footer selector和useBackgroundTaskNavigation共享使用
- selectedIPAgentIndex映射到此数组，三方必须保持排序一致

## 设计要点

1. **AsyncLocalStorage隔离**: 进程内队友在同Node进程中运行，但通过AsyncLocalStorage保持上下文隔离
2. **团队感知**: 支持agentName@teamName格式的身份标识
3. **空闲状态**: 队友可以处于空闲状态等待工作，无需重新启动
4. **消息队列**: pendingUserMessages支持用户向队友发送消息
5. **优雅关闭**: shutdownRequested标志支持优雅关闭而非强制kill

## 与其他文件的关系

- **导入**:
  - `isTerminalTaskStatus`, `SetAppState`, `Task`, `TaskStateBase` from Task
  - `Message` from types/message
  - `killInProcessTeammate` from spawnInProcess - 实际终止队友
  - `updateTaskState` from task/framework - 状态更新
  - `InProcessTeammateTaskState`, `appendCappedMessage`, `isInProcessTeammateTask` from ./types

- **被调用者**:
  - 团队管理相关组件
  - 队友转录视图
  - 消息路由系统

## 注意事项

1. **kill实现**: kill方法委托给killInProcessTeammate，实际终止逻辑在spawnInProcess模块
2. **消息注入**: injectUserMessageToTeammate仅在任务非终止状态下工作
3. **排序一致性**: getRunningTeammatesSorted的排序被多个组件共享，修改需谨慎
4. **空闲回调**: onIdleCallbacks用于领导者高效等待，无需轮询
