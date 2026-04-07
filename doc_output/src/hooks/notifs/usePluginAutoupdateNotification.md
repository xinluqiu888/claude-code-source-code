# usePluginAutoupdateNotification.tsx — 插件自动更新通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/usePluginAutoupdateNotification.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `usePluginAutoupdateNotification`
- **依赖**: React useState/useEffect, notifications, pluginAutoupdate

## 功能概述

本 Hook 监听插件自动更新事件，当插件被自动更新时显示通知，提示用户运行 `/reload-plugins` 应用更新。

## 核心内容详解

### 状态管理

```typescript
const [updatedPlugins, setUpdatedPlugins] = useState<string[]>([])
```

### 事件订阅

```typescript
useEffect(() => {
  if (getIsRemoteMode()) return
  
  const unsubscribe = onPluginsAutoUpdated(plugins => {
    logForDebugging(`Plugin autoupdate notification: ${plugins.length} plugin(s) updated`)
    setUpdatedPlugins(plugins)
  })
  
  return unsubscribe
}, [])
```

### 通知显示

```typescript
useEffect(() => {
  if (getIsRemoteMode()) return
  if (updatedPlugins.length === 0) return
  
  // 提取插件名（去掉 @marketplace 后缀）
  const pluginNames = updatedPlugins.map(id => {
    const atIndex = id.indexOf('@')
    return atIndex > 0 ? id.substring(0, atIndex) : id
  })
  
  // 构建显示名称
  const displayNames = pluginNames.length <= 2 
    ? pluginNames.join(' and ') 
    : `${pluginNames.length} plugins`
  
  addNotification({
    key: 'plugin-autoupdate-restart',
    jsx: <>
      <Text color="success">
        {pluginNames.length === 1 ? 'Plugin' : 'Plugins'} updated: {displayNames}
      </Text>
      <Text dimColor> · Run /reload-plugins to apply</Text>
    </>,
    priority: 'low',
    timeoutMs: 10000,
  })
}, [updatedPlugins, addNotification])
```

## 设计要点

### 1. 事件驱动

通过 `onPluginsAutoUpdated` 订阅自动更新事件，而非轮询。

### 2. 插件名提取

从 ID（如 "plugin@marketplace"）提取纯名称，更易读。

### 3. 批量显示

- 1-2 个插件：显示具体名称（"A and B"）
- 3+ 个插件：显示数量（"3 plugins"）

### 4. 低优先级

插件更新不紧急，使用 'low' 优先级。

### 5. 操作引导

明确提示运行 `/reload-plugins` 应用更新。

## 与其他文件的关系

- **pluginAutoupdate.ts**: 提供 `onPluginsAutoUpdated`
- **debug.ts**: 调试日志

## 注意事项

1. **重启需求**: 自动更新后需要手动 `/reload-plugins` 生效
2. **通知一次性**: 更新事件触发一次通知，不重复
3. **远程模式**: 远程模式下不显示插件更新通知
4. **调试日志**: 记录更新事件便于排查问题
