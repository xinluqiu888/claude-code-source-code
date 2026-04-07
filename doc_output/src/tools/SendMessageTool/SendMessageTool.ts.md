# SendMessageTool.ts — 代理间消息发送工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/SendMessageTool/SendMessageTool.ts`
- **作用**: 在代理团队成员之间发送消息

## 功能概述

该工具实现代理群（Agent Swarm）协议，支持向队友发送消息、广播到全队，以及处理关闭请求/响应、计划审批等结构化消息。支持跨会话通信（UDS和bridge模式）。

## 核心内容详解

### 输入Schema

```typescript
z.object({
  to: z.string().describe('Recipient: teammate name, "*" for broadcast, or address'),
  summary: z.string().optional().describe('5-10 word summary for UI preview'),
  message: z.union([
    z.string().describe('Plain text message content'),
    StructuredMessage(), // 结构化消息（关闭/审批）
  ]),
})
```

### 结构化消息类型

```typescript
z.discriminatedUnion('type', [
  z.object({ type: z.literal('shutdown_request'), reason: z.string().optional() }),
  z.object({ type: z.literal('shutdown_response'), request_id: z.string(), approve: semanticBoolean(), reason: z.string().optional() }),
  z.object({ type: z.literal('plan_approval_response'), request_id: z.string(), approve: semanticBoolean(), feedback: z.string().optional() }),
])
```

### 输出类型

支持多种输出类型：
- `MessageOutput`: 普通消息发送结果
- `BroadcastOutput`: 广播结果
- `RequestOutput`: 请求发送结果（如shutdown_request）
- `ResponseOutput`: 响应发送结果

### 核心处理逻辑

#### 消息发送处理

1. **Bridge模式**（UDS_INBOX特性）: 发送到远程控制会话
2. **UDS模式**: 发送到本地Unix Domain Socket
3. **本地代理恢复**: 如果目标代理已停止，自动恢复并发送消息
4. **普通消息**: 写入队友邮箱
5. **广播**: 向所有队友发送

#### 结构化消息处理

- **shutdown_request**: 发送关闭请求到指定代理
- **shutdown_response**: 处理关闭响应，批准后触发优雅关闭
- **plan_approval_response**: 处理计划审批响应

### 权限检查

```typescript
async checkPermissions(input, context): Promise<PermissionDecision>
```

- bridge地址需要显式用户确认（跨机器安全风险）
- 本地通信自动允许

### 输入验证

验证项：
1. `to`字段非空
2. 地址格式有效（bridge/uds模式）
3. 不包含`@`符号（单团队限制）
4. 纯文本消息需要summary
5. 结构化消息不能广播

## 设计要点

1. **协议支持**: 支持代理群协议（shutdown、plan approval）
2. **跨会话通信**: UDS和bridge模式支持跨会话/跨机器通信
3. **自动恢复**: 检测到停止的代理自动恢复
4. **权限控制**: 跨机器通信需要用户确认
5. **延迟执行**: `shouldDefer: true`

## 与其他文件的关系

- **teammateMailbox.ts**: 邮箱写入操作
- **peerSessions.ts**: bridge消息发送
- **udsClient.ts**: UDS消息发送
- **resumeAgent.ts**: 代理自动恢复
- **LocalAgentTask.ts**: 本地代理任务管理

## 注意事项

- 纯文本消息需要summary
- 结构化消息不能发送到`*`广播
- 队友只能看到自己的任务和消息
- 关闭批准后代理将优雅退出
