# directConnectManager.ts — 直连会话管理器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/server/directConnectManager.ts`
- **类型**: TypeScript 模块
- **作用**: 管理直连服务器的WebSocket连接和消息通信

## 功能概述

DirectConnectSessionManager类负责建立和维护与直连服务器的WebSocket连接，处理双向消息通信、权限请求和控制消息。它实现了客户端与服务器的实时通信机制。

## 核心内容详解

### 配置类型

```typescript
export type DirectConnectConfig = {
  serverUrl: string    // 服务器基础URL
  sessionId: string    // 会话ID
  wsUrl: string        // WebSocket URL
  authToken?: string   // 可选认证令牌
}
```

### 回调类型

```typescript
export type DirectConnectCallbacks = {
  onMessage: (message: SDKMessage) => void                    // 收到消息
  onPermissionRequest: (request: SDKControlPermissionRequest, requestId: string) => void  // 权限请求
  onConnected?: () => void                                    // 连接成功
  onDisconnected?: () => void                                // 连接断开
  onError?: (error: Error) => void                           // 发生错误
}
```

### 消息类型守卫

```typescript
function isStdoutMessage(value: unknown): value is StdoutMessage {
  return (
    typeof value === 'object' &&
    value !== null &&
    'type' in value &&
    typeof value.type === 'string'
  )
}
```

### DirectConnectSessionManager 类

#### 构造函数

```typescript
constructor(config: DirectConnectConfig, callbacks: DirectConnectCallbacks)
```

#### 方法

**`connect(): void`**
建立WebSocket连接：
1. 设置Authorization头（如果提供authToken）
2. 创建WebSocket连接
3. 绑定事件处理器

**事件处理**:
- **open**: 调用`onConnected`
- **message**: 解析每行JSON消息
  - `control_request`: 处理权限请求（`can_use_tool`）
  - 其他消息类型转发到`onMessage`
- **close**: 调用`onDisconnected`
- **error**: 调用`onError`

**`sendMessage(content: RemoteMessageContent): boolean`**
发送用户消息到服务器：
```typescript
{
  type: 'user',
  message: { role: 'user', content: content },
  parent_tool_use_id: null,
  session_id: '',
}
```

**`respondToPermissionRequest(requestId: string, result: RemotePermissionResponse): void`**
响应权限请求：
```typescript
{
  type: 'control_response',
  response: {
    subtype: 'success',
    request_id: requestId,
    response: {
      behavior: 'allow' | 'deny',
      updatedInput?: string,  // allow时
      message?: string,       // deny时
    },
  },
}
```

**`sendInterrupt(): void`**
发送中断信号取消当前请求：
```typescript
{
  type: 'control_request',
  request_id: crypto.randomUUID(),
  request: { subtype: 'interrupt' },
}
```

**`disconnect(): void`**
关闭WebSocket连接。

**`isConnected(): boolean`**
检查连接状态。

### 私有方法

**`sendErrorResponse(requestId: string, error: string): void`**
发送错误响应（用于不支持的control_request子类型）。

## 消息协议

### 从服务器接收

**控制请求** (`control_request`):
```typescript
{
  type: 'control_request',
  request_id: string,
  request: {
    subtype: 'can_use_tool',
    tool_name: string,
    tool_input: unknown,
  },
}
```

**SDK消息** (转发给回调): assistant、result、system等类型。

### 发送到服务器

**用户消息** (`user`): 用户输入内容。
**控制响应** (`control_response`): 权限请求响应。
**控制请求** (`control_request`): interrupt等控制命令。

## 设计要点

1. **多行消息处理**: 服务器可能发送多行JSON，按行分割处理
2. **错误响应**: 不支持的子类型发送错误响应防止服务器挂起
3. **异步解析**: 每行独立解析，失败继续处理下一行
4. **UUID生成**: 使用`crypto.randomUUID()`生成请求ID

## 与其他文件的关系

- **被 useDirectConnect.ts 使用**: React hook包装此管理器
- **被 REPL.tsx 使用**: 直连模式下的消息通信
- **使用 createDirectConnectSession.ts**: 获取config
- **使用 types.ts**: 共享类型定义
- **使用 entrypoints/sdk/controlTypes.ts**: SDK消息类型
- **使用 remote/RemoteSessionManager.ts**: RemotePermissionResponse类型

## 注意事项

1. **Headers兼容性**: Bun的WebSocket支持headers选项，但DOM类型定义不支持，使用`as unknown as string[]`转换
2. **消息过滤**: 过滤掉keep_alive、post_turn_summary等内部消息
3. **连接状态检查**: 发送前检查`readyState === WebSocket.OPEN`
4. **行分割**: 过滤空行处理
