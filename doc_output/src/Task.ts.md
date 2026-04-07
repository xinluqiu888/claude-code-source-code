# Task.ts — 任务系统核心类型与工具函数

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/Task.ts`
- **类型**: TypeScript 模块
- **导出内容**: `TaskType`、`TaskStatus`、`TaskHandle`、`TaskContext`、`TaskStateBase`、`Task`、`generateTaskId()`、`createTaskStateBase()`、`isTerminalTaskStatus()`
- **依赖关系**:
  - 导入: `crypto`, `AppState.js`, `ids.js`, `task/diskOutput.js`

## 功能概述

本文件定义了 Claude Code 任务系统的基础类型和工具函数。任务是 CLI 中异步操作的抽象，支持多种类型（Bash 命令、本地代理、远程代理、队友任务等）。每个任务有唯一ID、状态追踪、输出文件管理和生命周期控制。

## 核心内容详解

### 1. TaskType 联合类型 (第6-14行)

```typescript
export type TaskType =
  | 'local_bash'           // 本地 Bash 命令
  | 'local_agent'          // 本地子代理
  | 'remote_agent'         // 远程代理
  | 'in_process_teammate'  // 进程内队友
  | 'local_workflow'       // 本地工作流
  | 'monitor_mcp'          // MCP 监控
  | 'dream'                // Dream 任务
```

**前缀映射** (第79-87行):
```typescript
const TASK_ID_PREFIXES: Record<string, string> = {
  local_bash: 'b',           // 保持 'b' 向后兼容
  local_agent: 'a',
  remote_agent: 'r',
  in_process_teammate: 't',
  local_workflow: 'w',
  monitor_mcp: 'm',
  dream: 'd',
}
```

### 2. TaskStatus 联合类型 (第16-20行)

```typescript
export type TaskStatus =
  | 'pending'     // 待处理
  | 'running'     // 运行中
  | 'completed'   // 已完成
  | 'failed'      // 失败
  | 'killed'      // 已终止
```

### 3. isTerminalTaskStatus() 函数 (第27-29行)

检查任务是否处于终止状态：

```typescript
export function isTerminalTaskStatus(status: TaskStatus): boolean {
  return status === 'completed' || status === 'failed' || status === 'killed'
}
```

**用途**:
- 防止向已终止的队友注入消息
- 从 AppState 中驱逐已完成的任务
- 孤儿清理路径

### 4. TaskHandle 类型 (第31-34行)

任务句柄：

```typescript
export type TaskHandle = {
  taskId: string
  cleanup?: () => void  // 可选清理函数
}
```

### 5. SetAppState 类型 (第36行)

```typescript
export type SetAppState = (f: (prev: AppState) => AppState) => void
```

### 6. TaskContext 类型 (第38-42行)

任务执行上下文：

```typescript
export type TaskContext = {
  abortController: AbortController
  getAppState: () => AppState
  setAppState: SetAppState
}
```

### 7. TaskStateBase (第45-57行)

所有任务状态共享的基础字段：

```typescript
export type TaskStateBase = {
  id: string                   // 任务ID
  type: TaskType               // 任务类型
  status: TaskStatus           // 当前状态
  description: string          // 描述
  toolUseId?: string           // 关联的工具使用ID
  startTime: number            // 开始时间戳
  endTime?: number             // 结束时间戳
  totalPausedMs?: number       // 总暂停时间
  outputFile: string           // 输出文件路径
  outputOffset: number         // 输出偏移
  notified: boolean            // 是否已通知
}
```

### 8. LocalShellSpawnInput (第59-67行)

本地 shell 生成输入：

```typescript
export type LocalShellSpawnInput = {
  command: string
  description: string
  timeout?: number
  toolUseId?: string
  agentId?: AgentId
  kind?: 'bash' | 'monitor'  // UI 显示变体
}
```

### 9. Task 接口 (第72-76行)

多态任务接口：

```typescript
export type Task = {
  name: string
  type: TaskType
  kill(taskId: string, setAppState: SetAppState): Promise<void>
}
```

**设计说明** (第69-71行):
- `spawn` 和 `render` 从未被多态调用（已在 #22546 中移除）
- 所有六个 `kill` 实现只使用 `setAppState`
- `getAppState`/`abortController` 是死代码

### 10. generateTaskId() 函数 (第98-106行)

生成唯一任务ID：

```typescript
const TASK_ID_ALPHABET = '0123456789abcdefghijklmnopqrstuvwxyz'

export function generateTaskId(type: TaskType): string {
  const prefix = getTaskIdPrefix(type)
  const bytes = randomBytes(8)
  let id = prefix
  for (let i = 0; i < 8; i++) {
    id += TASK_ID_ALPHABET[bytes[i]! % TASK_ID_ALPHABET.length]
  }
  return id
}
```

**ID 生成算法**:
- 前缀：类型对应字母（b/a/r/t/w/m/d）
- 主体：8 个随机字符（0-9, a-z）
- 组合数：36^8 ≈ 2.8 万亿，足以抵抗暴力符号链接攻击

### 11. createTaskStateBase() 函数 (第108-125行)

创建任务状态基础对象：

```typescript
export function createTaskStateBase(
  id: string,
  type: TaskType,
  description: string,
  toolUseId?: string,
): TaskStateBase {
  return {
    id,
    type,
    status: 'pending',
    description,
    toolUseId,
    startTime: Date.now(),
    outputFile: getTaskOutputPath(id),
    outputOffset: 0,
    notified: false,
  }
}
```

**初始化值**:
- 状态：'pending'
- 开始时间：当前时间戳
- 输出文件：通过 `getTaskOutputPath(id)` 获取
- 输出偏移：0
- 已通知：false

## 设计要点

1. **类型安全**: 使用联合类型区分不同任务类型和状态
2. **唯一ID**: 基于密码学安全随机数生成
3. **输出管理**: 每个任务有独立的输出文件
4. **生命周期追踪**: 开始/结束时间、暂停时间、通知状态
5. **终止状态概念**: 明确定义哪些状态是终态
6. **简化接口**: Task 接口只保留 kill 方法

## 与其他文件的关系

- **tasks/**: 各任务类型实现（LocalAgentTask、RemoteAgentTask 等）
- **state/AppStateStore.ts**: AppState 包含 tasks 字典
- **state/teammateViewHelpers.ts**: 使用 isTerminalTaskStatus
- **utils/task/diskOutput.ts**: getTaskOutputPath 提供输出路径

## 使用场景

```typescript
// 创建新任务
const taskId = generateTaskId('local_bash')
const taskState = createTaskStateBase(taskId, 'local_bash', 'Running ls')

// 检查任务是否完成
if (isTerminalTaskStatus(taskState.status)) {
  // 清理资源
}

// 定义任务实现
const bashTask: Task = {
  name: 'Bash',
  type: 'local_bash',
  kill: async (taskId, setAppState) => {
    // 终止任务逻辑
  }
}
```

## 注意事项

1. **ID 格式**: 任务ID格式为 "前缀 + 8位随机字符"
2. **向后兼容**: local_bash 保持 'b' 前缀
3. **输出文件**: 通过 getTaskOutputPath 统一管理路径
4. **Task 接口简化**: 只有 kill 方法是多态必需的
5. **终止状态**: completed/failed/killed 三种终态
