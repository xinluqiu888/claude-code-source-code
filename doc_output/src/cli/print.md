# print.ts — Headless/非交互式模式核心执行模块

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/print.ts` |
| **文件类型** | TypeScript 核心模块 |
| **行数** | 约 2400+ 行 |
| **职责** | 实现无头（headless）模式下的查询执行、消息流处理和 SDK 集成 |

## 功能概述

本模块是 Claude Code CLI 在非交互式（headless/print）模式下的核心执行引擎。它负责处理 `--print` 模式下的完整会话生命周期，包括初始化、消息处理、工具调用、权限管理、MCP 服务器集成、状态同步等。

模块实现了 `runHeadless` 和 `runHeadlessStreaming` 两个主要函数，分别用于一次性执行和流式执行。它支持多种输出格式（JSON、stream-json、文本），处理会话恢复（resume）、继续（continue）、分支（fork）等场景，并集成了沙箱、MCP、插件系统、权限提示等高级功能。

该模块是连接 SDK 消费者（如 VS Code 扩展、CCR 远程控制）与 Claude Code 核心功能的桥梁，实现了完整的双向通信协议。

## 核心内容详解

### 导入（按功能分组）

#### 基础与平台
- `feature` from `bun:bundle` - 功能标志系统
- `readFile`, `stat` from `fs/promises` - 文件系统操作
- `dirname` from `path` - 路径处理

#### CLI 与 SDK
- `StructuredIO`, `RemoteIO` - IO 抽象层
- `Command`, `formatDescriptionWithSource`, `getCommandName` - 命令系统
- 各类 SDK 类型定义（`SDKStatus`, `ModelInfo`, `SDKMessage` 等）

#### 工具与 MCP
- `Tool`, `Tools`, `ToolPermissionContext` - 工具系统
- `assembleToolPool`, `filterToolsByDenyRules` - 工具池管理
- MCP 相关：`MCPServerConnection`, `McpSdkServerConfig`, `setupSdkMcpClients` 等

#### 状态与设置
- `AppState` - 应用状态
- `getSettings_DEPRECATED`, `getSettingsWithSources` - 设置管理
- `settingsChangeDetector`, `applySettingsChange` - 设置变更处理

#### 工具函数
- `gracefulShutdown`, `gracefulShutdownSync` - 优雅关闭
- `createIdleTimeoutManager` - 空闲超时
- `logEvent` - 分析日志
- `logForDebugging`, `logForDiagnosticsNoPII` - 调试日志

### 主要类型定义

#### `PromptValue`
```typescript
type PromptValue = string | ContentBlockParam[]
```
表示提示值可以是字符串或内容块数组。

### 核心函数

#### `joinPromptValues(values: PromptValue[]): PromptValue`
将多个提示值合并为一个。字符串以换行连接；如果有任何值是块数组，所有值都规范化为块并连接。

#### `canBatchWith(head: QueuedCommand, next: QueuedCommand | undefined): boolean`
判断 `next` 命令是否可以与 `head` 命令批量处理。只有提示模式的命令才能批量处理，且需要工作负载标签和 isMeta 标志匹配。

#### `runHeadless(...)`
主入口函数，负责：
- 初始化设置和配置
- 处理沙箱初始化
- 加载初始消息
- 执行主查询循环
- 处理输出格式（JSON、stream-json、文本）

#### `runHeadlessStreaming(...)`
流式执行函数，返回 `AsyncIterable<StdoutMessage>`，用于：
- 实现实时消息流
- 支持长时间运行的会话
- 与 SDK 消费者进行双向通信

#### `getStructuredIO(inputPrompt, options)`
创建并配置 StructuredIO 实例，根据配置选择 `RemoteIO` 或 `StructuredIO`。

#### `getCanUseToolFn(...)`
创建工具权限检查函数，根据配置选择不同的权限提示实现（stdio、synthetic、默认）。

#### `loadInitialMessages(...)`
加载会话的初始消息，处理：
- 新会话
- `--continue` 继续会话
- `--resume` 恢复会话
- `--teleport` 传送到特定会话点
- `--fork-session` 分支会话

#### `handleRewindFiles(...)`
处理 `--rewind-files` 功能，将文件系统恢复到特定消息点的状态。

### 内部函数（在 runHeadlessStreaming 中定义）

#### `run()`
主执行循环函数：
- 管理运行状态（running、runPhase）
- 处理命令队列（支持批量处理）
- 调用 QueryEngine 进行查询
- 处理权限提示和工具调用
- 管理会话生命周期

#### `drainCommandQueue()`
排空命令队列，将连续的提示模式命令批量处理为单个 `ask()` 调用。

#### `forwardMessagesToBridge()`
将新消息转发到桥接器（用于远程控制）。

#### `updateSdkMcp()`
更新 SDK MCP 客户端配置，处理服务器的添加和移除。

#### `registerElicitationHandlers(clients)`
在 MCP 客户端上注册请求处理程序，处理征求（elicitation）请求。

#### `applyMcpServerChanges(servers)`
应用 MCP 服务器变更，处理配置更新和连接管理。

#### `buildMcpServerStatuses()`
构建 MCP 服务器状态数组，用于控制响应。

#### `installPluginsAndApplyMcpInBackground()`
后台插件安装和 MCP 配置应用。

#### `refreshPluginState()`
刷新插件状态，重新加载命令和代理定义。

#### `applyPluginMcpDiff()`
应用插件 MCP 配置差异。

## 设计要点

1. **流式架构**：使用 `AsyncIterable` 实现真正的流式输出，支持长时间运行的会话
2. **批量处理**：将连续的提示命令批量处理，减少 API 调用次数
3. **状态管理**：通过闭包和可变引用（mutableMessages、sdkClients 等）管理跨调用状态
4. **插件热重载**：支持插件的动态安装和命令的热重载
5. **MCP 集成**：完整支持 MCP 服务器生命周期，包括连接、重连、配置更新
6. **权限系统**：集成多种权限提示模式（stdio、synthetic、默认）
7. **沙箱支持**：可选的沙箱初始化，支持网络权限委托
8. **会话恢复**：支持多种会话恢复场景（继续、恢复、分支、回溯）

## 与其他文件的关系

| 关系类型 | 文件 | 描述 |
|---------|------|------|
| **导入** | `src/cli/structuredIO.ts` | 使用 StructuredIO 进行结构化输入输出 |
| **导入** | `src/cli/remoteIO.ts` | 使用 RemoteIO 进行远程连接 |
| **导入** | `src/cli/ndjsonSafeStringify.ts` | 使用 NDJSON 安全序列化 |
| **导入** | `src/QueryEngine.ts` | 调用 ask() 进行查询 |
| **导入** | `src/services/mcp/client.ts` | MCP 客户端管理 |
| **导入** | `src/utils/sessionStorage.ts` | 会话状态持久化 |
| **导入** | `src/utils/gracefulShutdown.ts` | 优雅关闭 |
| **被导入** | `src/main.tsx` | 主入口调用 runHeadless |

## 注意事项

1. **内存管理**：`mutableMessages` 数组直接传递并被修改，存在内存累积风险；设置了最大消息 UUID 跟踪数（MAX_RECEIVED_UUIDS = 10,000）
2. **竞态条件**：`mcpChangesPromise` 用于序列化 MCP 服务器变更调用，防止并发竞争
3. **信号处理**：SIGINT 处理程序用于中止当前查询，然后优雅关闭
4. **插件安装**：支持同步（CLAUDE_CODE_SYNC_PLUGIN_INSTALL）和异步两种安装模式
5. **Bridge 模式**：当启用远程控制时，需要转发消息到桥接器
6. **空闲超时**：实现了空闲超时管理，防止资源浪费
7. **GC 优化**：在 Bun 环境下定期强制垃圾回收
8. **调试支持**：集成了 headless 性能分析器，用于诊断启动延迟
