# notifications.tsx — 通知队列管理系统

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/context/notifications.tsx`
- **类型**: React Hook
- **语言**: TypeScript + React

## 功能概述

提供完整的通知队列管理系统，支持优先级队列、通知折叠 (folding)、失效 (invalidation) 机制和自动超时处理。用于在应用中显示临时通知消息。

## 核心内容详解

### 类型定义

```typescript
type Priority = 'low' | 'medium' | 'high' | 'immediate'

interface BaseNotification {
  key: string
  invalidates?: string[]  // 被此通知失效的其他通知 key
  priority: Priority
  timeoutMs?: number
  fold?: (accumulator: Notification, incoming: Notification) => Notification
}

type TextNotification = BaseNotification & { text: string; color?: keyof Theme }
type JSXNotification = BaseNotification & { jsx: React.ReactNode }
```

### 主要导出

1. **useNotifications()** — 返回通知管理函数
   - `addNotification`: 添加通知到队列
   - `removeNotification`: 按 key 移除通知

2. **getNext(queue)** — 从队列中获取下一个最高优先级的通知

### 优先级系统

```typescript
const PRIORITIES: Record<Priority, number> = {
  immediate: 0,  // 最高优先级
  high: 1,
  medium: 2,
  low: 3,
}
```

## 设计要点

1. **立即优先级处理**:
   - 立即显示，中断当前通知
   - 清除现有超时并设置新的超时
   - 将当前通知重新排队 (如果它不是立即优先级)

2. **通知折叠 (Folding)**:
   - 当相同 key 的通知已存在时调用 fold 函数
   - 可以折叠到当前显示的通知或队列中的通知
   - 重置折叠后通知的超时

3. **失效机制**:
   - 新通知可以指定 invalidates 数组使其他通知失效
   - 被失效的通知从队列中移除
   - 如果当前显示的通知被失效，清除超时并显示下一个

4. **重复防止**:
   - 使用 Set 检查队列中是否已存在相同 key
   - 防止同一通知被重复添加

5. **队列处理**:
   - 当前通知超时后自动处理队列中的下一个
   - 通过 processQueue 递归处理
   - 挂载时检查初始状态中是否有通知需要处理

## 与其他文件的关系

- **AppState.tsx**: 使用 useAppStateStore 和 useSetAppState 管理通知状态
- **Theme**: 使用 Theme 类型定义文本通知的颜色

## 注意事项

1. 默认超时时间为 8000ms (8秒)
2. currentTimeoutId 是模块级变量，用于跟踪当前活动的超时
3. 使用 useCallback 缓存回调函数以避免不必要的重新渲染
4. 比较通知时使用 key 而不是引用，以处理重新创建的通知对象
5. 立即优先级通知不会将其他立即优先级通知重新排队
