# InProcessBackend.ts — 进程内后端实现

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/backends/InProcessBackend.ts`  
**关联模块**: 队友执行、任务系统  
**主要依赖**: `src/tasks/InProcessTeammateTask/InProcessTeammateTask.js`

## 功能概述

本文件实现用于进程内 teammates 的 `TeammateExecutor`：
- 与基于窗格的后端（tmux/iTerm2）不同，进程内 teammates 在同一 Node.js 进程中运行
- 通过 `AsyncLocalStorage` 进行上下文隔离
- 与领导者共享资源（API 客户端、MCP 连接）
- 通过 `AbortController` 终止（而非 `kill-pane`）

## 核心内容详解

### 类定义

```typescript
class InProcessBackend implements TeammateExecutor {
  readonly type = 'in-process'
  private context: ToolUseContext | null = null
}
```

### 核心方法

#### `setContext(context)`
设置 `ToolUseContext` 用于 AppState 访问。必须在 `spawn()` 前调用。

#### `isAvailable()`
进程内后端始终可用（无外部依赖），返回 `Promise.resolve(true)`。

#### `spawn(config)`
生成进程内 teammate：

1. 调用 `spawnInProcessTeammate()`：
   - 创建 `TeammateContext`
   - 创建独立的 `AbortController`
   - 在 `AppState.tasks` 中注册 teammate
   - 返回 spawn 结果

2. 如果成功，启动 agent 执行：
   - 调用 `startInProcessTeammate()`
   - 传递身份信息、任务 ID、提示、上下文等
   - 从父对话中剥离消息（避免内存泄漏）

#### `sendMessage(agentId, message)`
向进程内 teammate 发送消息：
- 解析 `agentId`（格式：`agentName@teamName`）
- 写入文件邮箱
- 所有 teammates（窗格和进程内）使用相同的邮箱机制

#### `terminate(agentId, reason)`
优雅地终止进程内 teammate：
- 查找 AppState 中的任务
- 检查是否已请求关闭
- 生成确定性请求 ID
- 创建关闭请求消息
- 写入 teammate 的邮箱
- 标记任务为已请求关闭

#### `kill(agentId)`
强制立即终止进程内 teammate：
- 查找 AppState 中的任务
- 调用 `killInProcessTeammate()` 取消执行
- 更新任务状态为 `'killed'`

#### `isActive(agentId)`
检查 teammate 是否仍然活跃：
- 查找 AppState 中的任务
- 检查任务状态是否为 `'running'`
- 检查 `AbortController` 是否未中止

## 设计要点

1. **上下文注入**: `setContext` 模式允许在生成前注入依赖
2. **统一邮箱**: 所有 teammates 使用相同的文件邮箱机制
3. **AbortController**: 用于取消而非 kill-pane
4. **优雅关闭**: `terminate` 发送请求，teammate 自行处理退出

## 与其他文件的关系

- **types.ts**: 实现 `TeammateExecutor` 接口
- **spawnInProcess.ts**: 使用 `spawnInProcessTeammate`, `killInProcessTeammate`
- **inProcessRunner.ts**: 使用 `startInProcessTeammate`
- **InProcessTeammateTask.ts**: 使用 `findTeammateTaskByAgentId`, `requestTeammateShutdown`
- **teammateMailbox.ts**: 使用 `writeToMailbox`, `createShutdownRequestMessage`

## 注意事项

- 必须在调用 `spawn()` 前调用 `setContext()`
- `terminate()` 是优雅的，等待 teammate 响应
- `kill()` 是强制的，立即中止
- 邮箱系统使用 agent 名称而非 agent ID 进行路由
- 返回的 `agentId` 格式为 `agentName@teamName`
