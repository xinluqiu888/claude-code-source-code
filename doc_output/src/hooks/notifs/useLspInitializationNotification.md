# useLspInitializationNotification.tsx — LSP 初始化通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useLspInitializationNotification.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `useLspInitializationNotification`
- **依赖**: React useState/useEffect/useCallback, useInterval, notifications

## 功能概述

本 Hook 轮询 LSP（Language Server Protocol）状态，当管理器初始化失败或任何 LSP 服务器进入错误状态时显示通知。

## 核心内容详解

### 轮询配置

```typescript
const LSP_POLL_INTERVAL_MS = 5000
```

每 5 秒轮询一次。

### 轮询控制

```typescript
const [shouldPoll, setShouldPoll] = useState(() => isEnvTruthy('true'))
```

使用惰性初始化避免每次渲染都计算，仅在 `ENABLE_LSP_TOOL` 设置时启用。

### 错误追踪

```typescript
const notifiedErrorsRef = useRef<Set<string>>(new Set())
```

使用 Set 追踪已通知的错误，避免重复通知。

### 错误处理流程

**1. 管理器初始化失败**
```typescript
if (status.status === 'failed') {
  addError('lsp-manager', status.error.message)
  setShouldPoll(false)  // 停止轮询
  return
}
```

**2. 服务器错误**
```typescript
const manager = getLspServerManager()
if (manager) {
  const servers = manager.getAllServers()
  for (const [serverName, server] of servers) {
    if (server.state === 'error' && server.lastError) {
      addError(serverName, server.lastError.message)
    }
  }
}
```

### 通知格式

```typescript
addNotification({
  key: `lsp-error-${source}`,
  jsx: <>
    <Text color="error">LSP for {displayName} failed</Text>
    <Text dimColor> · /plugin for details</Text>
  </>,
  priority: 'medium',
  timeoutMs: 8000,
})
```

同时添加到 `appState.plugins.errors` 供 `/doctor` 显示。

## 设计要点

### 1. 条件启用

通过环境变量控制是否启用轮询。

### 2. 错误去重

使用错误 key（source:message）去重，同一错误只通知一次。

### 3. 停止轮询

管理器初始化失败后停止轮询，减少资源消耗。

### 4. 滚动期间跳过

`getIsScrollDraining()` 为 true 时跳过轮询，避免与滚动帧竞争事件循环。

### 5. 提取显示名

从 source 如 "plugin:typescript-lsp:typescript" 提取 "typescript"。

## 与其他文件的关系

- **useInterval (usehooks-ts)**: 定时轮询
- **manager.ts**: LSP 管理器
- **config.ts**: 全局配置

## 注意事项

1. **性能考虑**: 5 秒轮询间隔，滚动期间跳过
2. **错误持久化**: 同时写入 appState 供 /doctor 查看
3. **多错误源**: 支持管理器错误和多个服务器错误
4. **远程模式**: 远程模式下不显示 LSP 通知
