# remoteIO.ts — 远程双向流 IO 实现

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/remoteIO.ts` |
| **文件类型** | TypeScript 类模块 |
| **行数** | 256 行 |
| **职责** | 为 SDK 模式提供支持会话跟踪的双向流式传输，支持 WebSocket/SSE 传输 |

## 功能概述

本模块实现 `RemoteIO` 类，继承自 `StructuredIO`，用于远程控制场景下的双向通信。它管理与远程服务器的连接，支持多种传输协议（WebSocket、SSE），并集成了 CCR（Claude Code Remote）v2 客户端功能。

该类负责：
- 建立和维护与远程服务器的连接
- 处理身份验证（Bearer Token）
- 实现心跳保活机制（bridge 模式）
- 支持会话状态恢复（CCR v2）
- 管理内部事件读写（用于会话持久化）
- 处理环境变量动态更新

`RemoteIO` 是连接 Claude Code CLI 与远程控制服务（如 claude.ai、CCR 基础设施）的关键组件。

## 核心内容详解

### 导入

| 导入项 | 来源 | 用途 |
|--------|------|------|
| `StdoutMessage` | `src/entrypoints/sdk/controlTypes.js` | SDK 输出消息类型 |
| `PassThrough` | `stream` | Node.js 流处理 |
| `URL` | `url` | URL 解析 |
| `getSessionId` | `../bootstrap/state.js` | 获取当前会话 ID |
| `getPollIntervalConfig` | `../bridge/pollConfig.js` | 获取轮询配置 |
| `registerCleanup` | `../utils/cleanupRegistry.js` | 注册清理回调 |
| `getSessionIngressAuthToken` | `../utils/sessionIngressAuth.js` | 获取会话认证令牌 |
| `CCRClient`, `CCRInitError` | `./transports/ccrClient.js` | CCR v2 客户端 |
| `SSETransport` | `./transports/SSETransport.js` | SSE 传输实现 |
| `ndjsonSafeStringify` | `./ndjsonSafeStringify.js` | NDJSON 安全序列化 |
| `StructuredIO` | `./structuredIO.js` | 父类 |

### 类定义

#### `RemoteIO extends StructuredIO`

**私有属性**：

| 属性名 | 类型 | 描述 |
|--------|------|------|
| `url` | `URL` | 远程服务器 URL |
| `transport` | `Transport` | 底层传输实例 |
| `inputStream` | `PassThrough` | 输入流（接收远程数据） |
| `isBridge` | `boolean` | 是否为 bridge 模式（从环境变量读取） |
| `isDebug` | `boolean` | 是否为调试模式 |
| `ccrClient` | `CCRClient \| null` | CCR v2 客户端实例（可选） |
| `keepAliveTimer` | `ReturnType<typeof setInterval> \| null` | 保活定时器 |

**构造函数**：

```typescript
constructor(
  streamUrl: string,
  initialPrompt?: AsyncIterable<string>,
  replayUserMessages?: boolean,
)
```

**构造流程**：
1. 创建 `PassThrough` 输入流并调用父类构造函数
2. 解析远程 URL
3. 准备请求头（包含会话令牌授权）
4. 创建动态刷新头信息的回调函数
5. 根据 URL 协议获取适当的传输实例
6. 设置数据传输回调（将数据写入输入流，bridge 调试模式下同时输出到 stdout）
7. 设置连接关闭回调（结束输入流触发优雅关闭）
8. 如果启用 CCR v2，初始化 `CCRClient`
9. 启动传输连接
10. 如果是 bridge 模式，启动保活定时器
11. 注册清理回调
12. 如果提供了初始提示，发送到输入流

**关键方法**：

#### `override flushInternalEvents(): Promise<void>`
刷新挂起的内部事件。如果 CCR 客户端存在，委托给它；否则返回已解决的 Promise。

#### `override get internalEventsPending(): number`
获取内部事件队列深度。如果 CCR 客户端存在，返回其队列深度；否则返回 0。

#### `async write(message: StdoutMessage): Promise<void>`
发送输出到传输层。

**行为**：
- 如果 CCR 客户端存在，通过 CCR 客户端写入事件
- 否则直接通过传输层写入
- 在 bridge 模式下，控制请求消息总是回显到 stdout，以便父进程检测权限请求

#### `close(): void`
优雅地清理连接：
- 清除保活定时器
- 关闭传输层连接
- 结束输入流

### CCR v2 初始化逻辑

当 `CLAUDE_CODE_USE_CCR_V2` 环境变量为真时：

1. **传输断言**：CCR v2 需要 SSE 传输，如果不是则抛出错误
2. **客户端创建**：创建 `CCRClient` 实例并初始化
3. **恢复状态**：保存恢复的工作者状态到 `restoredWorkerState`
4. **错误处理**：初始化失败时记录诊断日志并触发优雅关闭
5. **清理注册**：注册清理回调以关闭 CCR 客户端
6. **内部事件写入器**：注册用于会话持久化的内部事件写入器
7. **内部事件读取器**：注册用于会话恢复的内部事件读取器
8. **生命周期监听**：注册命令生命周期监听器，报告交付状态
9. **状态监听**：注册会话状态变更监听器
10. **元数据监听**：注册会话元数据变更监听器

### 保活机制

仅在 bridge 模式下启用：
- 间隔来自 GrowthBook 配置（`session_keepalive_interval_v2_ms`，默认 120 秒）
- 间隔为 0 时禁用
- 发送 `keep_alive` 类型消息到上游
- 防止 Envoy 等代理的闲置超时（#21931）
- 保活消息在到达客户端 UI 之前被过滤掉

## 设计要点

1. **传输抽象**：通过 `getTransportForUrl` 动态选择传输实现，支持 WebSocket 和 SSE
2. **身份验证刷新**：提供 `refreshHeaders` 回调，支持令牌动态刷新（父进程刷新后可通过文件或环境变量获取新令牌）
3. **延迟连接**：在所有回调都设置完成之后才启动连接，确保早期消息不会丢失
4. **Bridge 模式特殊处理**：在 bridge 模式下，控制请求消息总是回显到 stdout，以便父进程检测权限请求
5. **环境版本传递**：通过 `x-environment-runner-version` 头传递环境运行器版本
6. **错误隔离**：CCR 客户端初始化错误不会阻止整体流程，而是记录并继续

## 与其他文件的关系

| 关系类型 | 文件 | 描述 |
|---------|------|------|
| **继承** | `src/cli/structuredIO.ts` | 继承 StructuredIO 类 |
| **导入** | `src/cli/transports/ccrClient.ts` | 使用 CCRClient 进行 CCR v2 通信 |
| **导入** | `src/cli/transports/SSETransport.ts` | 使用 SSETransport 进行 SSE 通信 |
| **导入** | `src/cli/transports/transportUtils.ts` | 使用 getTransportForUrl 获取传输 |
| **导入** | `src/cli/ndjsonSafeStringify.ts` | 使用 NDJSON 安全序列化 |
| **导入** | `src/utils/sessionIngressAuth.ts` | 获取会话认证令牌 |
| **导入** | `src/utils/sessionState.ts` | 设置会话状态变更监听器 |
| **导入** | `src/utils/sessionStorage.ts` | 设置内部事件读写器 |
| **使用** | `src/main.tsx` | 在远程模式下创建 RemoteIO 实例 |

## 注意事项

1. **CCR v2 前置条件**：`CCRClient` 构造函数必须在 `transport.connect()` 之前同步运行，否则早期 SSE 帧的接收确认会被静默丢弃
2. **Bridge 模式依赖**：保活机制仅在 bridge 模式下启用，因为 BYOC 工作者有不同的网络路径
3. **令牌刷新**：动态刷新头信息回调允许父进程刷新令牌后无需重新创建 RemoteIO 实例
4. **清理顺序**：`close()` 方法按特定顺序清理资源：定时器 -> 传输 -> 输入流
5. **初始提示处理**：将初始提示异步写入输入流，处理块中可能已存在的尾随换行符以避免双换行问题
