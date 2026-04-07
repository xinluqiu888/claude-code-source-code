# useStartupNotification.ts — 启动通知通用 Hook

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useStartupNotification.ts`
- **类型**: React Hook
- **导出函数**: `useStartupNotification`
- **依赖**: React useEffect/useRef, notifications

## 功能概述

本 Hook 封装了在组件挂载时触发一次性通知的逻辑，统一处理远程模式门控和会话级只执行一次的保障。替代了之前分散在 10+ 个通知 Hook 中的手卷逻辑。

## 核心内容详解

### 参数类型

```typescript
type Result = Notification | Notification[] | null

function useStartupNotification(
  compute: () => Result | Promise<Result>
): void
```

### 执行流程

**1. 挂载检查**
```typescript
if (getIsRemoteMode() || hasRunRef.current) return
hasRunRef.current = true
```

**2. 异步计算**
```typescript
void Promise.resolve()
  .then(() => computeRef.current())
  .then(result => {
    if (!result) return
    for (const n of Array.isArray(result) ? result : [result]) {
      addNotification(n)
    }
  })
  .catch(logError)
```

### 返回值处理

| 返回值类型 | 行为 |
|------------|------|
| `null` | 跳过，不显示通知 |
| `Notification` | 显示单个通知 |
| `Notification[]` | 显示多个通知 |

## 设计要点

### 1. 单次执行

使用 `hasRunRef` 确保每会话只执行一次，不依赖 React 的严格模式。

### 2. 远程模式保护

自动检查远程模式，避免在远程模式下显示不适用的通知。

### 3. 计算函数稳定性

使用 `computeRef` 保存计算函数引用，避免依赖数组问题。

### 4. 错误处理

计算函数的错误被捕获并记录，不中断应用流程。

### 5. 支持异步

计算函数可以是同步或异步的，通过 `Promise.resolve()` 统一处理。

## 与其他文件的关系

- **state.ts**: 使用 `getIsRemoteMode`
- **notifications.ts**: 使用 `useNotifications`
- **其他通知 Hook**: 如 `useCanSwitchToExistingSubscription`, `useInstallMessages` 等

## 注意事项

1. **计算函数只执行一次**: 即使依赖变化也不会重新执行
2. **远程模式检查**: 在 Hook 内部检查，计算函数无需重复检查
3. **错误静默**: 错误被捕获并记录，不抛出
4. **返回值灵活性**: 支持单个或多个通知
