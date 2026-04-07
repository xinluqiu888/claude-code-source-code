# useTeammateShutdownNotification.ts — 队友生命周期通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useTeammateShutdownNotification.ts`
- **类型**: React Hook
- **导出函数**: `useTeammateLifecycleNotification`
- **依赖**: React useEffect/useRef, AppState, notifications

## 功能概述

本 Hook 在队友（Teammate/Agent）启动或关闭时触发批处理通知，使用 `fold()` 将重复事件合并为单个通知（如 "3 agents spawned"）。

## 核心内容详解

### 通知类型

**1. 启动通知 (spawn)**
```typescript
function makeSpawnNotif(count: number): Notification {
  return {
    key: 'teammate-spawn',
    text: count === 1 ? '1 agent spawned' : `${count} agents spawned`,
    priority: 'low',
    timeoutMs: 5000,
    fold: foldSpawn,
  }
}
```

**2. 关闭通知 (shutdown)**
```typescript
function makeShutdownNotif(count: number): Notification {
  return {
    key: 'teammate-shutdown',
    text: count === 1 ? '1 agent shut down' : `${count} agents shut down`,
    priority: 'low',
    timeoutMs: 5000,
    fold: foldShutdown,
  }
}
```

### 批处理逻辑

```typescript
function foldSpawn(acc: Notification, _incoming: Notification): Notification {
  return makeSpawnNotif(parseCount(acc) + 1)
}
```

使用 `fold()` 方法合并相同 key 的通知，计数器递增。

### 状态追踪

```typescript
const seenRunningRef = useRef<Set<string>>(new Set())
const seenCompletedRef = useRef<Set<string>>(new Set())
```

使用两个 Set 分别跟踪已见的运行中和已完成任务，避免重复通知。

### 检测逻辑

```typescript
for (const [id, task] of Object.entries(tasks)) {
  if (!isInProcessTeammateTask(task)) continue

  if (task.status === 'running' && !seenRunningRef.current.has(id)) {
    seenRunningRef.current.add(id)
    addNotification(makeSpawnNotif(1))
  }

  if (task.status === 'completed' && !seenCompletedRef.current.has(id)) {
    seenCompletedRef.current.add(id)
    addNotification(makeShutdownNotif(1))
  }
}
```

## 设计要点

### 1. 批处理机制

使用通知系统的 `fold()` 功能，快速连续的事件合并为一条通知。

### 2. Set 追踪

使用 Set 存储已见任务 ID，高效检测新事件。

### 3. 状态分离

运行中和已完成分别追踪，一个任务可能先触发 spawn 后触发 shutdown。

### 4. 低优先级

使用 'low' 优先级和 5 秒超时，不打扰用户主要工作。

### 5. 远程模式保护

非远程模式下才显示通知。

## 与其他文件的关系

- **types.ts**: 使用 `isInProcessTeammateTask` 类型守卫
- **notifications.ts**: 使用 `useNotifications`

## 注意事项

1. **快速启动/关闭**: 如果队友快速启动又关闭，可能只看到 shutdown 通知
2. **重启场景**: 同一 ID 的任务重启会被视为新事件
3. **内存泄漏**: 长期运行的会话可能累积大量已见 ID，但数量受限于任务总数
4. **计数解析**: 通过正则从通知文本中提取计数
