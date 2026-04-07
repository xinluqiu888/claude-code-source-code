# structuredIO.ts — SDK 结构化输入输出处理

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/structuredIO.ts` |
| **文件类型** | TypeScript 类模块 |
| **行数** | 860 行 |
| **职责** | 提供结构化读写 SDK 消息的方式，捕获 SDK 协议，处理权限请求和工具回调 |

## 功能概述

本模块实现 `StructuredIO` 类，是 Claude Code SDK 模式的核心输入输出处理层。它负责：
- 从输入流（如 stdin）解析结构化消息
- 处理各种 SDK 消息类型（user、assistant、system、control_request、control_response）
- 管理工具权限请求（can_use_tool）的生命周期
- 支持钩子回调（hook_callback）
- 处理 MCP 消息转发
- 实现征求（elicitation）请求处理
- 支持沙箱网络访问权限请求

该类提供了完整的双向通信机制，允许 SDK 消费者（如 VS Code 扩展）与 Claude Code 核心进行交互，包括请求工具使用权限、接收状态更新、发送用户消息等。

## 核心内容详解

### 导入

| 导入项 | 来源 | 用途 |
|--------|------|------|
| `ElicitResult`, `JSONRPCMessage` | `@modelcontextprotocol/sdk` | MCP 征求结果和消息类型 |
| `randomUUID` | `crypto` | UUID 生成 |
| `AssistantMessage` | `src/types/message.js` | 助手消息类型 |
| `HookInput`, `HookJSONOutput` | `src/entrypoints/agentSdkTypes.js` | SDK 钩子类型 |
| `SDKControlRequest`, `SDKControlResponse` | `src/entrypoints/sdk/controlTypes.js` | SDK 控制消息类型 |
| `Tool`, `ToolUseContext` | `src/Tool.js` | 工具相关类型 |
| `logForDebugging`, `logForDiagnosticsNoPII` | `src/utils/debug.js`, `src/utils/diagLogs.js` | 日志记录 |
| `hasPermissionsToUseTool` | `src/utils/permissions/permissions.js` | 权限检查 |
| `writeToStdout` | `src/utils/process.js` | 标准输出写入 |
| `ndjsonSafeStringify` | `./ndjsonSafeStringify.js` | NDJSON 安全序列化 |
| `Stream` | `../utils/stream.js` | 自定义流实现 |

### 常量

| 名称 | 类型 | 值 | 描述 |
|------|------|-----|------|
| `SANDBOX_NETWORK_ACCESS_TOOL_NAME` | `string` | `'SandboxNetworkAccess'` | 沙箱网络访问的合成工具名 |
| `MAX_RESOLVED_TOOL_USE_IDS` | `number` | `1000` | 已解决工具使用 ID 的最大跟踪数 |

### 类型定义

#### `PendingRequest<T>`
```typescript
type PendingRequest<T> = {
  resolve: (result: T) => void
  reject: (error: unknown) => void
  schema?: z.Schema
  request: SDKControlRequest
}
```
表示挂起的权限请求，包含解析/拒绝回调、可选的 Zod 模式和原始请求。

### 类定义

#### `StructuredIO`

**属性**：

| 属性名 | 类型 | 描述 |
|--------|------|------|
| `structuredInput` | `AsyncGenerator<StdinMessage \| SDKMessage>` | 结构化输入生成器 |
| `pendingRequests` | `Map<string, PendingRequest<unknown>>` | 挂起的请求映射（按 request_id） |
| `restoredWorkerState` | `Promise<SessionExternalMetadata \| null>` | CCR 外部元数据恢复状态 |
| `inputClosed` | `boolean` | 输入是否已关闭 |
| `unexpectedResponseCallback` | `function \| undefined` | 意外响应回调 |
| `resolvedToolUseIds` | `Set<string>` | 已解决的工具使用 ID 集合（防重复处理） |
| `prependedLines` | `string[]` | 预置的用户消息行（用于在下次消息前插入） |
| `onControlRequestSent` | `function \| undefined` | 控制请求发送回调 |
| `onControlRequestResolved` | `function \| undefined` | 控制请求解决回调 |
| `outbound` | `Stream<StdoutMessage>` | 出站消息流（FIFO 队列） |

**构造函数**：

```typescript
constructor(
  private readonly input: AsyncIterable<string>,
  private readonly replayUserMessages?: boolean,
)
```

**核心方法**：

#### `prependUserMessage(content: string): void`
在下次从 `this.input` 读取的消息之前队列一个用户回合。在迭代开始前和流中间都有效。

#### `private async *read()`
主要输入读取生成器。从输入流读取原始字符串，按换行分割，解析每行为结构化消息。

**处理流程**：
1. 处理 `prependedLines` 中预置的行
2. 从输入流读取块
3. 按换行符分割
4. 调用 `processLine` 解析每行
5. 输入关闭时，拒绝所有挂起的请求

#### `private async processLine(line: string): Promise<StdinMessage | SDKMessage | undefined>`
解析单行输入为结构化消息。

**支持的消息类型**：
- `keep_alive`：静默忽略
- `update_environment_variables`：直接应用到 `process.env`
- `control_response`：处理权限响应，解决挂起的请求
- `user`：用户消息
- `assistant`/`system`：助手/系统消息
- `control_request`：控制请求

#### `async write(message: StdoutMessage): Promise<void>`
将消息写入 stdout，使用 NDJSON 安全序列化。

#### `private async sendRequest<Response>(...): Promise<Response>`
发送控制请求并等待响应。使用 Promise + Map 存储挂起的请求，支持 AbortSignal 取消。

#### `createCanUseTool(onPermissionPrompt?): CanUseToolFn`
创建工具权限检查函数，是核心权限流程的实现。

**流程**：
1. 检查工具权限（hasPermissionsToUseTool）
2. 如果允许/拒绝，直接返回结果
3. 否则启动并行竞争：
   - 钩子评估（后台运行）
   - SDK 权限提示（通过 sendRequest 发送 can_use_tool 请求）
4. 谁先完成就使用谁的结果
5. 如果钩子先完成并做出决定，取消 SDK 请求

#### `createHookCallback(callbackId, timeout?): HookCallback`
创建钩子回调，允许 SDK 消费者为钩子提供响应。

#### `async handleElicitation(...): Promise<ElicitResult>`
发送征求请求到 SDK 消费者并等待响应。用于 MCP 服务器的用户交互。

#### `createSandboxAskCallback(): (hostPattern) => Promise<boolean>`
创建沙箱网络访问询问回调。将沙箱网络权限请求作为 can_use_tool 控制请求转发到 SDK 宿主。

#### `async sendMcpMessage(serverName, message): Promise<JSONRPCMessage>`
发送 MCP 消息到 SDK 服务器并等待响应。

#### `injectControlResponse(response): void`
注入控制响应以解决挂起的权限请求。由桥接器使用，将来自 claude.ai 的权限响应输入到 SDK 权限流。

#### `setOnControlRequestSent(callback): void`
注册 can_use_tool 控制请求写入 stdout 时的回调。由桥接器使用，将权限请求转发到 claude.ai。

#### `setOnControlRequestResolved(callback): void`
注册 can_use_tool control_response 到达时的回调。由桥接器使用，在 SDK 消费者赢得竞争时取消 claude.ai 上的陈旧权限提示。

### 辅助函数

#### `serializeDecisionReason(reason): string | undefined`
序列化权限决定原因为字符串。

#### `buildRequiresActionDetails(tool, input, toolUseID, requestId): RequiresActionDetails`
构建需要操作的详情对象，用于权限提示。

#### `executePermissionRequestHooksForSDK(...)`
执行权限请求钩子并返回决定（如果有）。返回 undefined 如果没有钩子做出决定。

## 设计要点

1. **防重复处理**：使用 `resolvedToolUseIds` Set 跟踪已解决的工具使用 ID，防止重复控制响应导致重复助手消息（会引发 API 400 错误）
2. **竞争模式**：`createCanUseTool` 实现钩子与 SDK 权限提示的并行竞争，谁先完成使用谁的结果
3. **预置消息**：`prependUserMessage` 允许在输入流中插入用户消息，支持会话恢复和钩子注入
4. **请求队列**：`outbound` Stream 确保 control_request 不会超越排队的 stream_events
5. **环境变量更新**：支持通过消息直接更新进程环境变量（用于桥接会话运行器的令牌刷新）
6. **生命周期管理**：`sendRequest` 管理请求的完整生命周期，包括取消和清理

## 与其他文件的关系

| 关系类型 | 文件 | 描述 |
|---------|------|------|
| **被继承** | `src/cli/remoteIO.ts` | RemoteIO 继承 StructuredIO |
| **导入** | `src/cli/ndjsonSafeStringify.ts` | 使用 NDJSON 安全序列化 |
| **导入** | `src/utils/permissions/permissions.js` | 使用 hasPermissionsToUseTool 检查权限 |
| **导入** | `src/utils/hooks.js` | 执行权限请求钩子 |
| **导入** | `src/utils/sessionState.ts` | 通知会话状态变更 |
| **导入** | `src/utils/commandLifecycle.ts` | 通知命令生命周期 |
| **使用** | `src/cli/print.ts` | 创建 StructuredIO/RemoteIO 实例用于 headless 模式 |

## 注意事项

1. **内存管理**：`resolvedToolUseIds` Set 限制大小为 1000，防止长时间会话中的内存无限增长
2. **竞态条件**：`sendRequest` 使用 Map 存储挂起的请求，确保并发请求的正确隔离
3. **输入关闭处理**：输入流关闭时，所有挂起的请求都会被拒绝
4. **意外响应**：通过 `unexpectedResponseCallback` 处理未知的 control_response（如桥接器场景）
5. **钩子取消**：钩子胜出时使用 `AbortController` 取消挂起的 SDK 请求
6. **Zod 验证**：响应可选地使用 Zod Schema 进行验证
7. **错误处理**：权限请求失败时返回拒绝决定而非抛出异常
