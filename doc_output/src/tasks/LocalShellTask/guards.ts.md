# guards.ts — LocalShellTask类型守卫

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/LocalShellTask/guards.ts`
- **类型**: TypeScript 类型定义文件
- **主要功能**: 提供LocalShellTask的状态类型定义和类型守卫

## 功能概述

本文件提取了LocalShellTask的状态类型和类型守卫函数，使非React消费者(如stopTask.ts via print.ts)无需引入React/ink模块即可使用这些类型。

## 核心内容详解

### 类型定义

#### `BashTaskKind`
Bash任务种类：
```typescript
type BashTaskKind = 'bash' | 'monitor'
```
- `'bash'`: 普通bash命令
- `'monitor'`: 监控任务，显示描述而非命令，使用不同的状态栏pill

#### `LocalShellTaskState`
本地Shell任务状态(继承TaskStateBase)：
```typescript
type LocalShellTaskState = TaskStateBase & {
  type: 'local_bash'          // 保持为'local_bash'以兼容持久化状态
  command: string             // 执行的命令
  result?: {
    code: number              // 退出码
    interrupted: boolean      // 是否被中断
  }
  completionStatusSentInAttachment: boolean
  shellCommand: ShellCommand | null
  unregisterCleanup?: () => void
  cleanupTimeoutId?: NodeJS.Timeout
  lastReportedTotalLines: number
  isBackgrounded: boolean     // false=前台运行, true=后台运行
  agentId?: AgentId           // 创建此任务的代理ID
  kind?: BashTaskKind         // UI显示变体
}
```

### 类型守卫

#### `isLocalShellTask`
检查对象是否为LocalShellTaskState：
```typescript
function isLocalShellTask(task: unknown): task is LocalShellTaskState
```
检查逻辑：
1. 是否为对象且不为null
2. 是否有'type'属性且值为'local_bash'

## 设计要点

1. **模块分离**: 将类型定义从LocalShellTask.tsx提取，避免非React消费者引入React/ink依赖
2. **向后兼容**: type保持为'local_bash'而非'local_shell'，兼容已持久化的会话状态
3. **Agent关联**: agentId字段用于在代理退出时终止孤儿bash任务
4. **行数跟踪**: lastReportedTotalLines用于计算增量通知

## 与其他文件的关系

- **导入**:
  - `TaskStateBase` from Task - 基础任务状态
  - `AgentId` from types/ids - 代理ID类型
  - `ShellCommand` from utils/ShellCommand - Shell命令类型

- **被导入**:
  - `LocalShellTask.tsx` - 主实现
  - `killShellTasks.ts` - 终止逻辑
  - `stopTask.ts` - 停止任务
  - `print.ts` - 打印输出(通过stopTask)

## 注意事项

1. **类型标识**: type字段值为'local_bash'而非'local_shell'，是为了兼容历史数据
2. **运行时字段**: unregisterCleanup和cleanupTimeoutId为运行时only
3. **Monitor区分**: kind='monitor'时显示描述而非命令，使用"Monitor details"对话框标题
4. **代理关联**: 当agentId存在时，代理退出会自动终止相关shell任务
