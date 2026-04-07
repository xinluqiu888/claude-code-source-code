# useDirectConnect.ts — 直连会话管理 Hook

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/useDirectConnect.ts`
- **类型**: React Hook
- **导出函数**: `useDirectConnect`
- **依赖**: React useEffect/useCallback/useMemo, DirectConnectSessionManager

## 功能概述

本 Hook 管理通过 WebSocket 直连 Claude Code 服务器的会话：
1. 建立和管理 WebSocket 连接
2. 处理服务器消息和权限请求
3. 支持发送消息和中断请求
4. 自动重连和错误处理

## 核心内容详解

### DirectConnectConfig

```typescript
type DirectConnectConfig = {
  wsUrl: string           // WebSocket URL
  sessionId: string       // 会话 ID
  token: string          // 认证令牌
}
```

### 连接生命周期

**建立连接**
```typescript
useEffect(() => {
  if (!config) return
  const manager = new DirectConnectSessionManager(config, handlers)
  manager.connect()
  return () => manager.disconnect()
}, [config])
```

### 消息处理

**onMessage**: 转换并添加消息到对话
```typescript
onMessage: sdkMessage => {
  if (isSessionEndMessage(sdkMessage)) setIsLoading(false)
  // 跳过重复的 init 消息
  const converted = convertSDKMessage(sdkMessage, { convertToolResults: true })
  if (converted.type === 'message') {
    setMessages(prev => [...prev, converted.message])
  }
}
```

**onPermissionRequest**: 创建权限确认队列项
```typescript
onPermissionRequest: (request, requestId) => {
  const tool = findToolByName(toolsRef.current, request.tool_name) ?? 
               createToolStub(request.tool_name)
  const syntheticMessage = createSyntheticAssistantMessage(request, requestId)
  const toolUseConfirm: ToolUseConfirm = { ... }
  setToolUseConfirmQueue(queue => [...queue, toolUseConfirm])
}
```

### 权限响应

| 用户操作 | 响应行为 |
|----------|----------|
| Allow | `behavior: 'allow'`, 更新输入 |
| Deny | `behavior: 'deny'`, 可选反馈消息 |
| Abort | `behavior: 'deny'`, 消息 "User aborted" |

### 连接断开处理

```typescript
onDisconnected: () => {
  if (!isConnectedRef.current) {
    // 从未连接成功 → 连接失败（如认证被拒绝）
    process.stderr.write(`\nFailed to connect to server at ${config.wsUrl}\n`)
  } else {
    // 已连接后断开 → 服务器退出或网络断开
    process.stderr.write('\nServer disconnected.\n')
  }
  void gracefulShutdown(1)
}
```

## 设计要点

### 1. 工具引用稳定性

使用 ref 保存 tools，避免 WebSocket 回调捕获陈旧闭包：
```typescript
const toolsRef = useRef(tools)
useEffect(() => { toolsRef.current = tools }, [tools])
```

### 2. 结果对象稳定性

使用 `useMemo` 保持返回对象引用稳定：
```typescript
return useMemo(
  () => ({ isRemoteMode, sendMessage, cancelRequest, disconnect }),
  [isRemoteMode, sendMessage, cancelRequest, disconnect]
)
```

### 3. Init 消息去重

服务器每轮发送 init 消息，客户端只处理第一次：
```typescript
if (sdkMessage.type === 'system' && sdkMessage.subtype === 'init') {
  if (hasReceivedInitRef.current) return
  hasReceivedInitRef.current = true
}
```

## 与其他文件的关系

- **DirectConnectSessionManager.ts**: WebSocket 连接管理
- **sdkMessageAdapter.ts**: SDK 消息转换
- **remotePermissionBridge.ts**: 权限请求桥接
- **gracefulShutdown.ts**: 优雅关闭处理

## 注意事项

1. **断开即退出**: 连接断开后调用 `gracefulShutdown(1)` 退出进程
2. **权限队列**: 权限请求加入队列，按顺序处理
3. **合成消息**: 权限请求创建合成助手消息展示给用户
4. **工具存根**: 未知工具使用 `createToolStub` 创建占位
