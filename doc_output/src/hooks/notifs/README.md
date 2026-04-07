# notifs/ — 通知 Hook 目录

> **一句话总结**：Claude Code 各种通知功能的 React Hooks 集合

---

## 目录职责

本目录包含各类通知相关的 React Hooks，负责在特定条件下向用户显示通知：
1. 启动时通知（安装、订阅、迁移等）
2. 运行时状态通知（IDE、MCP、LSP 等）
3. 权限模式通知（快速模式、自动模式）
4. 限制和配额通知（速率限制）
5. 生命周期通知（队友启动/关闭）

## 文件清单

### 启动通知

| 文件 | 职责简述 |
|------|---------|
| [useStartupNotification.ts](./useStartupNotification.ts.md) | 启动通知基础 Hook，封装单次执行和远程模式检查 |
| [useInstallMessages.tsx](./useInstallMessages.md) | 显示安装相关消息（路径、别名、错误） |
| [useCanSwitchToExistingSubscription.tsx](./useCanSwitchToExistingSubscription.md) | 提示用户可以使用现有 Claude 订阅 |
| [useModelMigrationNotifications.tsx](./useModelMigrationNotifications.md) | 显示模型迁移完成通知 |
| [useNpmDeprecationNotification.tsx](./useNpmDeprecationNotification.md) | 提示 npm 安装已弃用，建议使用原生安装器 |

### 运行时状态通知

| 文件 | 职责简述 |
|------|---------|
| [useIDEStatusIndicator.tsx](./useIDEStatusIndicator.md) | IDE 连接状态通知（连接/断开/错误） |
| [useMcpConnectivityStatus.tsx](./useMcpConnectivityStatus.md) | MCP 服务器连接失败和认证需求通知 |
| [useLspInitializationNotification.tsx](./useLspInitializationNotification.md) | LSP 服务器错误通知 |
| [useSettingsErrors.tsx](./useSettingsErrors.md) | 设置错误通知 |

### 权限和限制通知

| 文件 | 职责简述 |
|------|---------|
| [useAutoModeUnavailableNotification.ts](./useAutoModeUnavailableNotification.md) | 自动模式不可用时显示原因 |
| [useFastModeNotification.tsx](./useFastModeNotification.md) | 快速模式状态变化通知（组织变更、冷却期、超额拒绝） |
| [useRateLimitWarningNotification.tsx](./useRateLimitWarningNotification.md) | 速率限制警告和超额使用通知 |

### 生命周期通知

| 文件 | 职责简述 |
|------|---------|
| [useTeammateShutdownNotification.ts](./useTeammateShutdownNotification.md) | 队友启动和关闭的批处理通知 |
| [useDeprecationWarningNotification.tsx](./useDeprecationWarningNotification.md) | 模型弃用警告通知 |

### 插件通知

| 文件 | 职责简述 |
|------|---------|
| [usePluginAutoupdateNotification.tsx](./usePluginAutoupdateNotification.md) | 插件自动更新完成通知 |
| [usePluginInstallationStatus.tsx](./usePluginInstallationStatus.md) | 插件安装失败通知 |

## 设计模式

### 1. useStartupNotification 模式

大多数启动通知使用 `useStartupNotification` 基础 Hook：
```typescript
export function useSomeNotification() {
  useStartupNotification(async () => {
    // 检查条件
    if (shouldSkip) return null
    // 返回通知或通知数组
    return { key: '...', text: '...', priority: 'low' }
  })
}
```

### 2. 远程模式保护

大多数通知检查 `getIsRemoteMode()`，远程模式下不显示。

### 3. 次数限制

部分通知使用全局配置限制显示次数（如订阅提示最多 3 次）。

### 4. 批处理和折叠

支持 `fold()` 方法的通知可以合并重复事件（如队友启动通知）。

### 5. 状态追踪

使用 ref 或 state 追踪已显示的通知，避免重复。

## 与其他目录的关系

```
notifs/
    ↓ 依赖
services/ (MCP, LSP, OAuth, limits)
utils/ (config, auth, billing, settings)
state/ (AppState)
context/ (notifications)
```
