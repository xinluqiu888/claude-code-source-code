# useFastModeNotification.tsx — 快速模式通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useFastModeNotification.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `useFastModeNotification`
- **依赖**: React useEffect, AppState, fastMode utils

## 功能概述

本 Hook 监听快速模式（Fast Mode）的各种状态变化并显示相应通知：
1. 组织启用/禁用快速模式
2. 快速模式因超额使用被拒绝
3. 快速模式进入/退出冷却期

## 核心内容详解

### 通知键

```typescript
const COOLDOWN_STARTED_KEY = 'fast-mode-cooldown-started'
const COOLDOWN_EXPIRED_KEY = 'fast-mode-cooldown-expired'
const ORG_CHANGED_KEY = 'fast-mode-org-changed'
const OVERAGE_REJECTED_KEY = 'fast-mode-overage-rejected'
```

### 1. 组织状态变化

```typescript
onOrgFastModeChanged(orgEnabled => {
  if (orgEnabled) {
    addNotification({
      key: ORG_CHANGED_KEY,
      color: 'fastMode',
      text: 'Fast mode is now available · /fast to turn on',
    })
  } else if (isFastMode) {
    setAppState(prev => ({ ...prev, fastMode: false }))
    addNotification({
      key: ORG_CHANGED_KEY,
      color: 'warning',
      text: 'Fast mode has been disabled by your organization',
    })
  }
})
```

### 2. 超额拒绝

```typescript
onFastModeOverageRejection(message => {
  setAppState(prev => ({ ...prev, fastMode: false }))
  addNotification({
    key: OVERAGE_REJECTED_KEY,
    color: 'warning',
    text: message,
  })
})
```

### 3. 冷却期

**冷却开始**
```typescript
onCooldownTriggered((resetAt, reason) => {
  const resetIn = formatDuration(resetAt - Date.now(), { hideTrailingZeros: true })
  const message = getCooldownMessage(reason, resetIn)
  addNotification({
    key: COOLDOWN_STARTED_KEY,
    invalidates: [COOLDOWN_EXPIRED_KEY],  // 互斥
    text: message,
    color: 'warning',
  })
})
```

**冷却结束**
```typescript
onCooldownExpired(() => {
  addNotification({
    key: COOLDOWN_EXPIRED_KEY,
    invalidates: [COOLDOWN_STARTED_KEY],  // 互斥
    color: 'fastMode',
    text: 'Fast limit reset · now using fast mode',
  })
})
```

### 冷却原因消息

| 原因 | 消息 |
|------|------|
| 'overloaded' | Fast mode overloaded and is temporarily unavailable · resets in {time} |
| 'rate_limit' | Fast limit reached and temporarily disabled · resets in {time} |

## 设计要点

### 1. 事件订阅模式

使用 `onXxx` 回调订阅状态变化，返回取消订阅函数。

### 2. 互斥通知

使用 `invalidates` 字段确保冷却开始和结束通知互斥。

### 3. 状态同步

组织禁用或超额拒绝时，同步更新 AppState 中的 fastMode。

### 4. 时间格式化

使用 `formatDuration` 格式化冷却时间，隐藏尾随零。

### 5. 条件渲染

快速模式未启用时跳过冷却期监听。

## 与其他文件的关系

- **fastMode.ts**: 提供 `isFastModeEnabled`, `onOrgFastModeChanged`, `onFastModeOverageRejection`, `onCooldownTriggered`, `onCooldownExpired`
- **format.ts**: 提供 `formatDuration`

## 注意事项

1. **订阅清理**: 组件卸载时取消事件订阅
2. **远程模式**: 远程模式下不显示快速模式通知
3. **immediate 优先级**: 快速模式相关通知使用 immediate 优先级
4. **超额处理**: 超额拒绝自动关闭快速模式
