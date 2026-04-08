# types.ts — InProcessTeammateTask类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/InProcessTeammateTask/types.ts`
- **类型**: TypeScript 类型定义文件
- **主要功能**: 定义进程内队友(InProcessTeammate)任务的类型和工具函数

## 功能概述

本文件定义了InProcessTeammateTaskState类型及相关类型，用于管理在同Node.js进程中运行的队友代理。与后台代理不同，进程内队友使用AsyncLocalStorage进行隔离，支持团队感知身份和计划模式审批流程。

## 核心内容详解

### 核心类型

#### `TeammateIdentity`
队友身份标识(存储在任务状态中)：
```typescript
type TeammateIdentity = {
  agentId: string        // 如 "researcher@my-team"
  agentName: string      // 如 "researcher"
  teamName: string
  color?: string
  planModeRequired: boolean
  parentSessionId: string // 领导者会话ID
}
```

#### `InProcessTeammateTaskState`
进程内队友任务状态(继承TaskStateBase)：
- `type`: 'in_process_teammate' - 类型标识
- `identity`: TeammateIdentity - 队友身份信息
- `prompt`: string - 执行提示
- `model?: string` - 可选模型覆盖
- `selectedAgent?: AgentDefinition` - 可选代理定义
- `abortController`: 终止整个队友
- `currentWorkAbortController`: 终止当前回合(不终止队友)
- `awaitingPlanApproval`: boolean - 计划模式审批跟踪
- `permissionMode`: PermissionMode - 权限模式
- `messages?`: Message[] - 对话历史(用于缩放视图)
- `inProgressToolUseIDs?`: Set<string> - 正在执行的工具ID(用于动画)
- `pendingUserMessages`: string[] - 待发送的用户消息队列
- `isIdle`: boolean - 是否空闲
- `shutdownRequested`: boolean - 是否请求关闭
- `onIdleCallbacks`: 空闲回调(运行时 only)

### 常量

#### `TEAMMATE_MESSAGES_UI_CAP`
UI消息上限：50条

根据BQ分析(2026-03-20)，在500+回合会话中每个代理占用约20MB RSS，并发代理爆发时达到125MB。Whale会话(9a990de8)在2分钟内启动了292个代理，达到36.8GB。主要成本是这个数组持有每条消息的第二个完整副本。

### 工具函数

#### `isInProcessTeammateTask`
类型守卫函数，检查对象是否为InProcessTeammateTaskState。

#### `appendCappedMessage`
追加消息到数组并限制大小：
- 如果prev为空，返回包含item的新数组
- 如果达到上限，移除最旧的条目后追加
- 始终返回新数组(AppState不可变性要求)

## 设计要点

1. **内存优化**: UI消息限制为50条，完整对话存储在allMessages数组和磁盘转录文件中
2. **双AbortController设计**: 
   - `abortController`: 终止整个队友生命周期
   - `currentWorkAbortController`: 仅终止当前回合，保持队友存活
3. **运行时vs持久化分离**: AsyncLocalStorage引用不序列化到磁盘
4. **空闲状态管理**: 支持队友空闲等待工作，通过回调机制通知领导者

## 与其他文件的关系

- **导入**:
  - `TaskStateBase` from Task - 基础任务状态
  - `AgentToolResult` from AgentTool - 代理工具结果类型
  - `AgentDefinition` from loadAgentsDir - 代理定义
  - `PermissionMode` - 权限模式
  - `AgentProgress` from LocalAgentTask - 代理进度

- **导出**:
  - 被 `InProcessTeammateTask.tsx` 使用
  - 被其他需要检查队友任务类型的模块使用

## 注意事项

1. **消息数组限制**: task.messages仅用于缩放转录对话框，不需要完整上下文
2. **运行时字段**: abortController、unregisterCleanup、onIdleCallbacks为运行时only，不序列化到磁盘
3. **团队感知**: 支持agentName@teamName格式的团队感知身份
4. **邮箱分离**: Mailbox消息存储在teamContext.inProcessMailboxes，不在task.messages中
