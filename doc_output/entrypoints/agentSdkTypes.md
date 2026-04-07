# agentSdkTypes.ts — Agent SDK 类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/entrypoints/agentSdkTypes.ts`
- **类型**: TypeScript 类型定义文件
- **语言**: TypeScript

## 功能概述

Claude Code Agent SDK 的主入口文件，重新导出公共 SDK API。包含核心类型、运行时类型、控制协议类型和工具类型，供 SDK 消费者和构建者使用。

## 核心内容详解

### 导出分类

1. **MCP SDK 类型** (来自 @modelcontextprotocol/sdk):
   - `CallToolResult`
   - `ToolAnnotations`

2. **控制协议类型** (@alpha):
   - `SDKControlRequest`
   - `SDKControlResponse`
   - 从 `./sdk/controlTypes.js` 导入

3. **核心类型**:
   - 从 `./sdk/coreTypes.js` 导入
   - 可序列化的通用类型 (消息、配置)

4. **运行时类型**:
   - 从 `./sdk/runtimeTypes.js` 导入
   - 不可序列化类型 (回调、接口)

5. **设置类型**:
   - `Settings` 类型 (从生成的 JSON Schema 生成)

6. **工具类型**:
   - 从 `./sdk/toolTypes.js` 导入
   - 标记为 @internal 直到 SDK API 稳定

### 函数导出 (占位实现)

大多数函数目前抛出 "not implemented" 错误，实际实现在 SDK 运行时提供：

1. **tool()** — 创建工具定义
2. **createSdkMcpServer()** — 创建 MCP 服务器实例
3. **AbortError** — 中止错误类
4. **query()** — 查询函数
5. **unstable_v2_createSession()** — V2 API: 创建持久会话
6. **unstable_v2_resumeSession()** — V2 API: 恢复现有会话
7. **unstable_v2_prompt()** — V2 API: 单次提示便利函数
8. **getSessionMessages()** — 获取会话消息
9. **listSessions()** — 列出会话
10. **getSessionInfo()** — 获取会话信息
11. **renameSession()** — 重命名会话
12. **tagSession()** — 标记会话
13. **forkSession()** — 分叉会话

### 内部类型 (Daemon 原语)

1. **CronTask** — 定时任务
2. **CronJitterConfig** — Cron 调度调优配置
3. **ScheduledTaskEvent** — 定时任务事件 (fire/missed)
4. **ScheduledTasksHandle** — 定时任务句柄
5. **InboundPrompt** — 来自 claude.ai 的用户消息
6. **ConnectRemoteControlOptions** — 远程控制连接选项
7. **RemoteControlHandle** — 远程控制句柄

## 设计要点

1. **分层导出**:
   - SDK 消费者使用 coreSchemas.ts
   - SDK 构建者使用 controlTypes.ts
   - 清晰的分离避免类型污染

2. **占位实现模式**:
   - 类型定义在 SDK 包中
   - 实际实现由 CLI 运行时注入
   - 抛出错误防止直接调用

3. **版本控制**:
   - V2 API 标记为 @alpha 和 unstable
   - 内部类型明确标记
   - 工具类型标记为 @internal

4. **远程控制支持**:
   - 完整的远程控制类型定义
   - WebSocket 连接管理
   - 消息流处理

## 与其他文件的关系

- **./sdk/coreTypes.js**: 核心可序列化类型
- **./sdk/runtimeTypes.js**: 运行时类型
- **./sdk/controlTypes.js**: 控制协议类型
- **./sdk/toolTypes.js**: 工具类型
- **./sdk/settingsTypes.generated.js**: 生成的设置类型

## 注意事项

1. 函数在 SDK 中是占位实现，实际功能由 CLI 提供
2. V2 API 不稳定，可能变化
3. 工具类型标记为内部，不应依赖
4. 远程控制功能需要 OAuth 认证
5. 控制协议用于 SDK 实现与 CLI 之间的通信
