# client.ts — MCP 客户端

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/mcp/client.ts`
- **所属模块**: MCP Service
- **功能类型**: MCP 服务器连接和工具调用

## 功能概述

该模块实现 Model Context Protocol (MCP) 客户端功能，支持连接 MCP 服务器、列出/调用工具、处理认证和错误恢复。

## 核心内容详解

### 错误类

#### `McpAuthError`
认证错误（401），需要重新认证。

#### `McpSessionExpiredError`
会话过期（404 + JSON-RPC code -32001）。

#### `McpToolCallError`
工具调用返回错误结果（isError: true）。

### 主要函数

#### `isMcpSessionExpiredError(error): boolean`
检测会话过期错误。
- HTTP 404
- 错误消息包含 `"code":-32001`

#### `getMcpToolTimeoutMs(): number`
获取 MCP 工具超时（默认 ~27.8 小时）。

#### MCP 连接管理
- `ensureConnectedClient` — 确保客户端已连接
- `disconnectMcpServer` — 断开服务器
- `callMcpTool` — 调用 MCP 工具
- `listMcpTools` — 列出可用工具

### 传输类型支持

- **stdio** — 标准输入/输出
- **sse** — 服务器发送事件
- **http** — HTTP 流式
- **ws** — WebSocket
- **sdk** — SDK 控制

### 认证缓存

`MCP_AUTH_CACHE_TTL_MS = 15 * 60 * 1000`（15 分钟）

**功能：**
- 缓存需要认证的服务器
- 文件：`~/.claude/mcp-needs-auth-cache.json`
- 去重并发认证请求

## 设计要点

1. **多传输支持** — 灵活连接各种 MCP 服务器
2. **认证恢复** — 401 时标记需要重新认证
3. **会话过期检测** — 特定错误码识别
4. **工具超时** — 可配置，默认极长
5. **缓存优化** — 减少重复认证请求

## 与其他文件的关系

- **被调用**: 工具执行系统
- **依赖**: `@modelcontextprotocol/sdk`
- **关联**: `auth.ts` 认证处理

## 注意事项

- 会话过期后需要重新获取客户端
- 401 错误会缓存到文件，避免重复认证弹窗
- 工具描述长度限制 2048 字符
