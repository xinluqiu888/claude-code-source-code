# usePluginInstallationStatus.tsx — 插件安装状态通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/usePluginInstallationStatus.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `usePluginInstallationStatus`
- **依赖**: React useEffect/useMemo, AppState, notifications

## 功能概述

本 Hook 监听插件安装状态，当有市场或插件安装失败时显示通知。

## 核心内容详解

### 状态提取

```typescript
const installationStatus = useAppState(s => s.plugins.installationStatus)

const { totalFailed, failedMarketplacesCount, failedPluginsCount } = useMemo(() => {
  if (!installationStatus) {
    return {
      totalFailed: 0,
      failedMarketplacesCount: 0,
      failedPluginsCount: 0,
    }
  }
  
  const failedMarketplaces = installationStatus.marketplaces.filter(m => m.status === 'failed')
  const failedPlugins = installationStatus.plugins.filter(p => p.status === 'failed')
  
  return {
    totalFailed: failedMarketplaces.length + failedPlugins.length,
    failedMarketplacesCount: failedMarketplaces.length,
    failedPluginsCount: failedPlugins.length,
  }
}, [installationStatus])
```

### 通知逻辑

```typescript
useEffect(() => {
  if (getIsRemoteMode()) return
  if (!installationStatus) {
    logForDebugging('No installation status to monitor')
    return
  }
  if (totalFailed === 0) return
  
  logForDebugging(
    `Plugin installation status: ${failedMarketplacesCount} failed marketplaces, ${failedPluginsCount} failed plugins`
  )
  
  addNotification({
    key: 'plugin-install-failed',
    jsx: <>
      <Text color="error">
        {totalFailed} {plural(totalFailed, 'plugin')} failed to install
      </Text>
      <Text dimColor> · /plugin for details</Text>
    </>,
    priority: 'medium',
  })
}, [addNotification, totalFailed, failedMarketplacesCount, failedPluginsCount, installationStatus])
```

## 设计要点

### 1. 使用 useMemo

缓存失败计数，避免不必要的 effect 触发。

### 2. 调试日志

记录详细状态便于排查问题。

### 3. 单数复数适配

使用 `plural()` 工具函数处理 "plugin" 的单复数。

### 4. /plugin 引导

引导用户使用 `/plugin` 查看详情。

### 5. 中等优先级

安装失败影响功能，但不阻断主流程，使用 'medium' 优先级。

## 与其他文件的关系

- **AppState.ts**: 读取插件安装状态
- **stringUtils.ts**: 提供 `plural`
- **debug.ts**: 调试日志

## 注意事项

1. **状态可能为 null**: 初始时可能没有安装状态
2. **重复通知**: 每次依赖变化都会尝试添加通知，依赖通知系统的去重
3. **计数聚合**: 市场失败和插件失败合并统计
4. **远程模式**: 远程模式下不显示插件安装通知
5. **调试信息**: 详细记录市场失败数和插件失败数
