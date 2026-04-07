# useAutoModeUnavailableNotification.ts — 自动模式不可用通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useAutoModeUnavailableNotification.ts`
- **类型**: React Hook
- **导出函数**: `useAutoModeUnavailableNotification`
- **依赖**: React useEffect/useRef, AppState, notifications

## 功能概述

本 Hook 在 Shift+Tab 循环经过自动模式位置时显示一次性通知，告知用户自动模式不可用的原因（设置、熔断器、组织白名单等）。

## 核心内容详解

### 触发条件

当满足以下条件时显示通知：
1. `TRANSCRIPT_CLASSIFIER` 功能已启用
2. 非远程模式
3. 通知未显示过（`shownRef`）
4. 用户通过 Shift+Tab 从非默认模式循环回默认模式（跳过自动模式位置）

**循环检测逻辑**
```typescript
const wrappedPastAutoSlot =
  mode === 'default' &&
  prevMode !== 'default' &&
  prevMode !== 'auto' &&
  !isAutoModeAvailable &&
  hasAutoModeOptIn()
```

### 通知内容

```typescript
addNotification({
  key: 'auto-mode-unavailable',
  text: getAutoModeUnavailableNotification(reason),
  color: 'warning',
  priority: 'medium',
})
```

**可能的原因**
- 设置问题
- 熔断器触发
- 组织白名单限制

### 状态追踪

使用 `prevModeRef` 跟踪模式变化，检测循环方向。

## 设计要点

### 1. 一次性通知

使用 `shownRef` 确保每会话只显示一次，避免重复打扰。

### 2. 启动时降级

启动时的自动模式降级（defaultMode: auto 静默降级）由 `verifyAutoModeGateAccess` 和 `checkAndDisableAutoModeIfNeeded` 处理，不在此 Hook 中。

### 3. 循环检测

仅当用户显式循环经过自动模式位置时才触发，而非任何模式变化都触发。

## 与其他文件的关系

- **permissionSetup.ts**: 提供 `getAutoModeUnavailableReason` 和 `getAutoModeUnavailableNotification`
- **settings.ts**: 提供 `hasAutoModeOptIn`

## 注意事项

1. **与启动降级的区分**: 此 Hook 处理的是运行时循环触发，启动降级在其他地方处理
2. **原因多样性**: 支持多种自动模式不可用的原因
3. **非远程模式**: 远程模式下不显示此通知
