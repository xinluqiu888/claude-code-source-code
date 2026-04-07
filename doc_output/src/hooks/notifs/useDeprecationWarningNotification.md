# useDeprecationWarningNotification.tsx — 模型弃用警告通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useDeprecationWarningNotification.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `useDeprecationWarningNotification`
- **依赖**: React useEffect/useRef, notifications, model deprecation

## 功能概述

本 Hook 在检测到当前使用的模型已被弃用时显示警告通知，且只在警告内容变化时显示（避免重复显示相同警告）。

## 核心内容详解

### 参数

```typescript
function useDeprecationWarningNotification(model: string): void
```

### 核心逻辑

```typescript
useEffect(() => {
  if (getIsRemoteMode()) return
  
  const deprecationWarning = getModelDeprecationWarning(model)
  
  // 只有当警告内容变化时才显示
  if (deprecationWarning && deprecationWarning !== lastWarningRef.current) {
    lastWarningRef.current = deprecationWarning
    addNotification({
      key: 'model-deprecation-warning',
      text: deprecationWarning,
      color: 'warning',
      priority: 'high',
    })
  }
  
  // 如果没有警告，重置追踪
  if (!deprecationWarning) {
    lastWarningRef.current = null
  }
}, [model, addNotification])
```

### 状态追踪

```typescript
const lastWarningRef = useRef<string | null>(null)
```

使用 ref 追踪上次显示的警告内容，只在内容变化时触发新通知。

## 设计要点

### 1. 变化检测

通过比较新旧警告内容，避免重复显示相同警告。

### 2. 远程模式保护

远程模式下不显示弃用警告。

### 3. 高优先级

模型弃用是重要信息，使用 'high' 优先级。

### 4. 重置机制

当模型切换为非弃用模型时，重置追踪状态，为下次显示做准备。

## 与其他文件的关系

- **deprecation.ts**: 提供 `getModelDeprecationWarning`
- **notifications.ts**: 使用 `useNotifications`

## 注意事项

1. **警告缓存**: `lastWarningRef` 在组件生命周期内保持，切换模型后会重置
2. **远程模式**: 远程会话的模型弃用由服务端处理
3. **警告来源**: 弃用警告来自模型配置，可能包含迁移建议
