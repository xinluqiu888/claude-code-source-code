# useInstallMessages.tsx — 安装消息通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useInstallMessages.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `useInstallMessages`
- **依赖**: useStartupNotification, nativeInstaller

## 功能概述

本 Hook 在启动时检查安装状态，显示相关的安装消息（路径问题、别名设置、错误等）。

## 核心内容详解

### 优先级映射

| 条件 | 优先级 |
|------|--------|
| `type === 'error'` 或 `userActionRequired` | 'high' |
| `type === 'path'` 或 `type === 'alias'` | 'medium' |
| 其他 | 'low' |

### 颜色映射

| 类型 | 颜色 |
|------|------|
| 'error' | 'error' |
| 其他 | 'warning' |

### 处理流程

```typescript
useStartupNotification(async () => {
  const messages = await checkInstall()
  return messages.map((message, index) => {
    let priority: 'low' | 'medium' | 'high' | 'immediate' = 'low'
    if (message.type === 'error' || message.userActionRequired) {
      priority = 'high'
    } else if (message.type === 'path' || message.type === 'alias') {
      priority = 'medium'
    }
    
    return {
      key: `install-message-${index}-${message.type}`,
      text: message.message,
      priority,
      color: message.type === 'error' ? 'error' : 'warning',
    }
  })
})
```

## 设计要点

### 1. 启动时检查

利用 `useStartupNotification` 在应用启动时检查，避免运行时干扰。

### 2. 批量通知

返回通知数组，支持同时显示多条安装消息。

### 3. 动态优先级

根据消息类型和是否需要用户操作动态确定优先级。

### 4. 唯一键

使用索引和类型组合生成唯一 key，避免重复。

## 与其他文件的关系

- **useStartupNotification.ts**: 基础 Hook
- **nativeInstaller/index.ts**: 提供 `checkInstall`

## 注意事项

1. **异步检查**: `checkInstall` 可能涉及文件系统检查
2. **多条消息**: 可能同时返回多条不同类型的消息
3. **用户操作**: `userActionRequired` 标记需要用户关注的消息
4. **索引依赖**: 使用数组索引作为 key 的一部分，顺序变化可能影响通知去重
