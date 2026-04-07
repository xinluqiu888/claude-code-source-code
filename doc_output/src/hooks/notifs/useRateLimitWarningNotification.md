# useRateLimitWarningNotification.tsx — 速率限制警告通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useRateLimitWarningNotification.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `useRateLimitWarningNotification`
- **依赖**: React useState/useEffect/useRef/useMemo, notifications, Claude AI limits

## 功能概述

本 Hook 监听 Claude AI 使用限制，在达到限制或使用超额模式时显示警告通知。

## 核心内容详解

### 参数

```typescript
function useRateLimitWarningNotification(model: string): void
```

### 状态计算

```typescript
const rateLimitWarning = useMemo(
  () => getRateLimitWarning(claudeAiLimits, model),
  [claudeAiLimits, model]
)

const usingOverageText = useMemo(
  () => getUsingOverageText(claudeAiLimits),
  [claudeAiLimits]
)
```

使用 `useMemo` 避免不必要的重新计算。

### 超额通知

```typescript
const [hasShownOverageNotification, setHasShownOverageNotification] = useState(false)

useEffect(() => {
  if (getIsRemoteMode()) return
  
  if (
    claudeAiLimits.isUsingOverage && 
    !hasShownOverageNotification &&
    (!isTeamOrEnterprise || hasBillingAccess)
  ) {
    addNotification({
      key: 'limit-reached',
      text: usingOverageText,
      priority: 'immediate',
    })
    setHasShownOverageNotification(true)
  } else if (!claudeAiLimits.isUsingOverage && hasShownOverageNotification) {
    setHasShownOverageNotification(false)
  }
}, [...])
```

### 速率限制警告

```typescript
useEffect(() => {
  if (getIsRemoteMode()) return
  
  if (rateLimitWarning && rateLimitWarning !== shownWarningRef.current) {
    shownWarningRef.current = rateLimitWarning
    addNotification({
      key: 'rate-limit-warning',
      jsx: <Text><Text color="warning">{rateLimitWarning}</Text></Text>,
      priority: 'high',
    })
  }
}, [rateLimitWarning, addNotification])
```

## 设计要点

### 1. 状态追踪

使用 `hasShownOverageNotification` 确保超额通知每会话只显示一次。

### 2. 变化检测

使用 `shownWarningRef` 追踪速率限制警告内容，只在内容变化时显示新通知。

### 3. 组织账户处理

Team/Enterprise 账户在没有账单访问权限时不显示超额通知。

### 4. 优先级区分

- 超额通知：immediate 优先级
- 速率限制警告：high 优先级

### 5. 重置逻辑

当不再使用超额模式时重置状态，为下次通知做准备。

## 与其他文件的关系

- **claudeAiLimits.ts**: 提供 `getRateLimitWarning`, `getUsingOverageText`
- **claudeAiLimitsHook.ts**: 提供 `useClaudeAiLimits`
- **auth.ts**: 提供 `getSubscriptionType`
- **billing.ts**: 提供 `hasClaudeAiBillingAccess`

## 注意事项

1. **团队/企业账户**: 有特殊的账单访问控制逻辑
2. **警告去重**: 相同内容的警告不会重复显示
3. **超额一次性**: 超额通知每会话只显示一次
4. **模型依赖**: 速率限制警告可能因模型而异
5. **远程模式**: 远程模式下不显示限制通知
