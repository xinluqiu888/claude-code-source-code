# killShellTasks.ts — LocalShellTask终止辅助函数

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/LocalShellTask/killShellTasks.ts`
- **类型**: TypeScript 模块
- **主要功能**: 提供纯(非React)终止辅助函数用于LocalShellTask

## 功能概述

本文件提取了终止LocalShellTask的辅助函数，使runAgent.ts可以在不引入React/Ink模块图的情况下终止代理范围内的bash任务。

## 核心内容详解

### 主要函数

#### `killTask`
终止单个任务：
```typescript
function killTask(taskId: string, setAppState: SetAppStateFn): void
```

执行流程：
1. 通过updateTaskState更新任务状态
2. 验证任务正在运行且为LocalShellTask
3. 调用shellCommand.kill()终止进程
4. 调用shellCommand.cleanup()清理资源
5. 调用unregisterCleanup()注销清理处理器
6. 清除cleanupTimeoutId(如果存在)
7. 设置状态为'killed'，endTime为当前时间
8. 异步驱逐任务输出(evictTaskOutput)

错误处理：
- kill和cleanup调用包裹在try-catch中
- 错误通过logError记录，不中断流程

#### `killShellTasksForAgent`
终止指定代理创建的所有运行中bash任务：
```typescript
function killShellTasksForAgent(
  agentId: AgentId,
  getAppState: () => AppState,
  setAppState: SetAppStateFn
): void
```

使用场景：
- 在runAgent.ts的finally块中调用
- 防止后台进程比启动它们的代理存活更久
- 避免"10-day fake-logs.sh zombies"类问题

执行流程：
1. 遍历AppState中的所有任务
2. 筛选出符合条件的任务：
   - 是LocalShellTask
   - agentId匹配指定代理
   - 状态为'running'
3. 对每个匹配任务调用killTask
4. 清除所有发送给该代理的队列通知(dequeueAllMatching)

清理通知队列：
- killTask会异步触发'killed'通知
- 清除已排队的通知和后续可能到达的通知
- 无消费者匹配dead agentId时，通知无害停留

## 设计要点

1. **非React实现**: 纯函数实现，不依赖React/Ink
2. **防御性编程**: 所有操作包裹错误处理
3. **孤儿进程防护**: 代理退出时自动清理其shell任务
4. **通知清理**: 防止通知堆积和内存泄漏

## 与其他文件的关系

- **导入**:
  - `AppState` from state/AppState
  - `AgentId` from types/ids
  - `logForDebugging`, `logError` - 日志记录
  - `dequeueAllMatching` from messageQueueManager - 消息队列管理
  - `evictTaskOutput` from task/diskOutput - 输出驱逐
  - `updateTaskState` from task/framework - 状态更新
  - `isLocalShellTask` from ./guards - 类型守卫

- **被调用者**:
  - `runAgent.ts` - 代理退出时清理bash任务
  - `LocalShellTask.tsx` - kill方法委托

## 注意事项

1. **异步驱逐**: evictTaskOutput是异步操作，不阻塞状态更新
2. **日志记录**: 调试日志和错误日志分开处理
3. **队列清理**: 清除代理相关的排队通知，避免死代理的消息堆积
4. **孤儿任务**: 设计用于防止代理退出后shell任务继续运行成为僵尸进程
