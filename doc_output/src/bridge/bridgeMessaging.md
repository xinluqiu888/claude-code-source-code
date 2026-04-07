# bridgeMessaging.ts — 桥接消息传输层共享工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/bridge/bridgeMessaging.ts`
- **文件类型**: TypeScript 模块
- **行数**: 约 462 行
- **主要职责**: 提供桥接层的消息处理工具函数，包括类型守卫、入站消息路由、服务器控制请求处理和消息去重机制

## 功能概述

`bridgeMessaging.ts` 是桥接功能的核心传输层模块，负责处理 WebSocket 消息的解析、路由和响应。该模块为基于环境的桥接核心（`initBridgeCore`）和无环境桥接核心（`initEnvLessBridgeCore`）提供共享的消息处理基础设施。

该模块包含多个关键功能：类型守卫函数用于验证消息类型；入站消息处理逻辑负责解析服务器消息并路由到相应的处理器；服务器控制请求处理机制用于响应服务器发起的会话生命周期事件（如初始化、设置模型、中断等）；以及一个基于循环缓冲区实现的 UUID 去重集合，用于防止消息回显和重复交付。

所有函数都是纯函数，不依赖闭包状态，所有协作者（传输层、会话ID、UUID集合、回调函数）都作为参数传递，使得模块可以被多个不同的桥接实现复用。

## 核心内容详解

### 导入模块

| 导入 | 用途 |
|------|------|
| `randomUUID` | 生成结果消息的唯一标识符 |
| `SDKMessage` / `SDKControlRequest` / `SDKControlResponse` | SDK 消息类型定义 |
| `SDKResultSuccess` | 成功结果消息类型 |
| `logEvent` | 分析事件记录 |
| `EMPTY_USAGE` | 空使用量常量 |
| `Message` | 内部消息类型 |
| `normalizeControlMessageKeys` | 控制消息键规范化 |
| `logForDebugging` | 调试日志记录 |
| `stripDisplayTagsAllowEmpty` | 去除显示标签 |
| `errorMessage` | 错误消息提取 |
| `PermissionMode` | 权限模式类型 |
| `jsonParse` | JSON 安全解析 |
| `ReplBridgeTransport` | 传输层接口类型 |

### 类型守卫函数

#### `isSDKMessage(value: unknown): value is SDKMessage`
验证值是否为 SDKMessage 类型，检查非空、对象类型和 `type` 属性存在性。

#### `isSDKControlResponse(value: unknown): value is SDKControlResponse`
验证值是否为控制响应消息，检查 `type === 'control_response'` 和 `response` 属性存在。

#### `isSDKControlRequest(value: unknown): value is SDKControlRequest`
验证值是否为控制请求消息，检查 `type === 'control_request'`、`request_id` 和 `request` 属性存在。

#### `isEligibleBridgeMessage(m: Message): boolean`
判断消息是否适合转发到桥接传输层。排除虚拟消息，仅允许 `user`、`assistant` 类型和 `local_command` 子类型的系统消息。

#### `extractTitleText(m: Message): string | undefined`
从消息中提取可用于会话标题的文本内容。过滤非用户消息、元消息、工具结果、压缩摘要和非人工来源的消息。

### 入站消息处理

#### `handleIngressMessage(...)`
处理来自服务器的入站 WebSocket 消息：
- 解析 JSON 并规范化控制消息键
- 检查是否为控制响应消息（权限响应）
- 检查是否为控制请求消息（服务器发起的请求）
- 使用类型守卫验证 SDKMessage
- 通过 UUID 集合进行回显和重复交付检测
- 将用户消息转发到 `onInboundMessage` 回调

### 服务器控制请求处理

#### `ServerControlRequestHandlers` 类型
定义处理服务器控制请求所需的协作者集合：
- `transport`: 传输层实例
- `sessionId`: 会话ID
- `outboundOnly`: 出站模式标志
- `onInterrupt`: 中断回调
- `onSetModel`: 设置模型回调
- `onSetMaxThinkingTokens`: 设置最大思考令牌回调
- `onSetPermissionMode`: 设置权限模式回调

#### `handleServerControlRequest(request, handlers)`
响应服务器发起的控制请求，支持的子类型：
- `initialize`: 返回最小能力集（命令列表为空，输出样式为 normal）
- `set_model`: 调用模型设置回调
- `set_max_thinking_tokens`: 调用思考令牌设置回调
- `set_permission_mode`: 调用权限模式设置回调，根据回调结果返回成功或错误响应
- `interrupt`: 调用中断回调
- 未知类型：返回错误响应（防止服务器挂起）

在出站模式下，除 `initialize` 外的所有可变请求都返回错误响应。

### 结果消息构建

#### `makeResultMessage(sessionId: string): SDKResultSuccess`
构建用于会话归档的最小成功结果消息，包含：
- 类型、子类型、持续时间、成本、使用量等字段
- 空使用量（`EMPTY_USAGE`）
- 随机生成的 UUID
- 关联的会话ID

### BoundedUUIDSet 类

基于循环缓冲区实现的 FIFO 有界集合，用于消息去重。

**构造函数**: `constructor(capacity: number)`
- 设置容量上限
- 初始化环形数组和 Set

**方法**:
- `add(uuid: string): void` - 添加 UUID，如果已存在则跳过；如果容量已满则淘汰最旧的条目
- `has(uuid: string): boolean` - 检查 UUID 是否存在于集合中
- `clear(): void` - 清空集合并重置索引

**实现细节**:
- 使用 `writeIdx` 循环索引写入位置
- `set` 用于 O(1) 查找
- 当容量达到上限时，自动淘汰 `ring[writeIdx]` 位置的旧条目

## 设计要点

1. **纯函数设计**: 所有函数不依赖闭包状态，参数化所有依赖，便于测试和复用
2. **防御性编程**: 严格的类型守卫和运行时检查，防止服务器挂起
3. **UUID 去重**: 双重保护机制（`recentPostedUUIDs` 和 `recentInboundUUIDs`）处理回显和重放场景
4. **服务器超时防护**: 控制请求必须在 ~10-14 秒内响应，否则会断开连接
5. **出站模式支持**: 允许桥接以仅出站模式运行，拒绝入站控制操作
6. **错误透明性**: 出站模式下返回明确的错误信息而非虚假成功

## 与其他文件的关系

| 文件 | 关系描述 |
|------|----------|
| `replBridgeTransport.ts` | 导入 `ReplBridgeTransport` 类型用于处理服务器控制请求 |
| `types.ts` | 使用 `Message` 类型进行消息过滤和标题提取 |
| `../entrypoints/agentSdkTypes.js` | 导入 SDK 消息类型定义 |
| `../entrypoints/sdk/controlTypes.js` | 导入控制请求/响应类型 |
| `../services/analytics/index.js` | 记录 `tengu_bridge_message_received` 分析事件 |
| `../utils/controlMessageCompat.js` | 规范化控制消息键 |
| `../utils/displayTags.js` | 去除显示标签以提取纯净文本 |
| `bridgeMain.ts` / `replBridge.ts` | 调用本模块的函数处理消息和控制请求 |

## 注意事项

1. **服务器超时风险**: 控制请求必须在约 10-14 秒内响应，否则服务器会关闭 WebSocket 连接。实现中所有分支都会返回响应，不存在无响应路径。

2. **UUID 去重限制**: `BoundedUUIDSet` 的容量是有限的（默认为 2000），超旧的 UUID 可能被淘汰。这在极端长会话或消息风暴场景下可能导致重复处理。

3. **出站模式行为**: 当 `outboundOnly` 为 true 时，除 `initialize` 外的所有控制请求都返回错误。这是有意的设计，用于支持仅出站桥接场景。

4. **异步回调**: `onInboundMessage` 回调是 fire-and-forget 风格（使用 `void` 调用），不等待完成。这对于附件解析等异步操作很重要。

5. **错误吞没**: 消息解析错误会被捕获并记录到调试日志，不会向上传播。这意味着格式错误的消息会被静默丢弃。

6. **类型守卫局限性**: `isSDKMessage` 仅检查 `type` 属性是否为字符串，更具体的类型窄化需要调用者进一步处理。
