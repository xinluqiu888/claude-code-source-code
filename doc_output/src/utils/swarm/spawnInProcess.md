# spawnInProcess.ts — 进程内队友启动

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/spawnInProcess.ts`  
**关联模块**: 任务系统、Agent 上下文  
**主要依赖**: `src/state/AppState.js`, `src/Task.js`, `src/utils/abortController.js`

## 功能概述

本文件实现进程内（in-process）teammate 的创建和注册：
- 与基于进程的 teammates（tmux/iTerm2）不同，进程内 teammates 在同一 Node.js 进程中运行
- 使用 `AsyncLocalStorage` 进行上下文隔离
- 在 AppState 中注册 `InProcessTeammateTask`

## 核心内容详解

### 类型定义

```typescript
type SpawnContext = {
  setAppState: SetAppStateFn  // 用于注册任务的 AppState setter
  toolUseId?: string           // 可选的工具使用 ID
}

type InProcessSpawnConfig = {
  name: string                 // 显示名称，如 "researcher"
  teamName: string             // 所属团队
  prompt: string               // 初始提示/任务
  color?: string               // 可选 UI 颜色
  planModeRequired: boolean    // 是否要求计划模式
  model?: string               // 可选模型覆盖
}

type InProcessSpawnOutput = {
  success: boolean
  agentId: string              // 格式: "name@team"
  taskId?: string              // AppState 中的任务 ID
  abortController?: AbortController
  teammateContext?: ReturnType<typeof createTeammateContext>
  error?: string
}
```

### 核心函数

#### `spawnInProcessTeammate(config, context)`
启动进程内 teammate：
1. 生成确定性 agent ID（`formatAgentId(name, teamName)`）
2. 创建独立的 `AbortController`
3. 获取父会话 ID 用于关联
4. 创建 teammate 身份对象
5. 创建 `AsyncLocalStorage` 上下文
6. 注册 Perfetto trace agent
7. 创建任务状态并注册到 AppState
8. 注册清理处理器

#### `killInProcessTeammate(taskId, setAppState)`
终止进程内 teammate：
- 通过 `abortController.abort()` 取消执行
- 更新任务状态为 `'killed'`
- 从 `teamContext.teammates` 中移除
- 从团队文件中移除成员
- 清理 Perfetto agent 注册
- 触发 `emitTaskTerminatedSdk`

## 设计要点

1. **独立 AbortController**: Teammates 不应在领导查询中断时被中止
2. **Perfetto 追踪**: 注册 agent 以在层级视图中可视化
3. **状态管理**: 任务状态存储在 AppState.tasks 中，key 为 taskId
4. **清理处理**: 使用 `registerCleanup` 确保优雅关闭

## 与其他文件的关系

- **teammateContext.ts**: 创建 `AsyncLocalStorage` 上下文
- **InProcessTeammateTask**: 使用 `runWithTeammateContext` 执行 agent 循环
- **task/framework.ts**: 使用 `registerTask`, `evictTerminalTask`
- **abortController.ts**: 创建独立的 `AbortController`

## 注意事项

- `spawnInProcessTeammate` 返回后，实际 agent 执行由 `InProcessTeammateTask` 组件驱动
- 任务状态包含 `abortController`，允许外部终止
- `STOPPED_DISPLAY_MS` 后自动驱逐终端任务
