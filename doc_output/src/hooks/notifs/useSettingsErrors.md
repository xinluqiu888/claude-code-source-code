# useSettingsErrors.tsx — 设置错误通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useSettingsErrors.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `useSettingsErrors`
- **返回值**: `ValidationError[]`
- **依赖**: React useState/useEffect, notifications, settings validation

## 功能概述

本 Hook 监听设置变化，当检测到设置错误时显示通知提示用户使用 `/doctor` 查看详情。

## 核心内容详解

### 常量

```typescript
const SETTINGS_ERRORS_NOTIFICATION_KEY = 'settings-errors'
```

### 状态管理

```typescript
const [errors, setErrors] = useState<ValidationError[]>(() => {
  const { errors } = getSettingsWithAllErrors()
  return errors
})
```

使用惰性初始化避免重复计算。

### 设置变化监听

```typescript
const handleSettingsChange = useCallback(() => {
  const { errors } = getSettingsWithAllErrors()
  setErrors(errors)
}, [])

useSettingsChange(handleSettingsChange)
```

### 通知逻辑

```typescript
useEffect(() => {
  if (getIsRemoteMode()) return
  
  if (errors.length > 0) {
    const message = `Found ${errors.length} settings ${
      errors.length === 1 ? 'issue' : 'issues'
    } · /doctor for details`
    addNotification({
      key: SETTINGS_ERRORS_NOTIFICATION_KEY,
      text: message,
      color: 'warning',
      priority: 'high',
      timeoutMs: 60000,
    })
  } else {
    removeNotification(SETTINGS_ERRORS_NOTIFICATION_KEY)
  }
}, [errors, addNotification, removeNotification])
```

## 设计要点

### 1. 实时监听

通过 `useSettingsChange` 监听设置变化，实时更新错误状态。

### 2. 单例通知

使用固定 key，新错误出现时更新通知而非创建新通知。

### 3. 自动清除

错误修复后自动移除通知。

### 4. 长超时

60 秒超时，给用户足够时间看到并响应。

### 5. 高优先级

设置错误影响功能使用，使用 'high' 优先级。

## 与其他文件的关系

- **useSettingsChange.ts**: 设置变化监听 Hook
- **allErrors.ts**: 提供 `getSettingsWithAllErrors`
- **validation.ts**: 验证错误类型定义

## 注意事项

1. **错误汇总**: 显示错误数量而非详细列表
2. **单复数适配**: 根据错误数量使用 "issue" 或 "issues"
3. **/doctor 引导**: 引导用户使用 /doctor 查看详情
4. **返回值**: 返回错误数组供组件使用
5. **远程模式**: 远程模式下不显示设置错误通知
