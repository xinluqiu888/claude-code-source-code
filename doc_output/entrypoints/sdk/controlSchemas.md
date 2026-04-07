# controlSchemas.ts — SDK 控制协议 Schema

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/entrypoints/sdk/controlSchemas.ts`
- **类型**: TypeScript Schema 定义
- **语言**: TypeScript

## 功能概述

定义 SDK 控制协议的 Zod Schema，用于 SDK 实现与 CLI 之间的通信。供 SDK 构建者 (如 Python SDK) 使用，与 CLI 进程通信。

## 核心内容详解

### 外部类型占位

**JSONRPCMessagePlaceholder** — 来自 @modelcontextprotocol/sdk 的 JSONRPCMessage
- 视为 unknown 类型
- MCP 消息的占位符

### Hook 回调类型

**SDKHookCallbackMatcherSchema**:
- `matcher`: 匹配器字符串 (可选)
- `hookCallbackIds`: Hook 回调 ID 数组
- `timeout`: 超时 (可选)

### 控制请求类型

1. **SDKControlInitializeRequestSchema** — 初始化请求
   - `subtype`: 'initialize'
   - `hooks`: Hook 配置记录
   - `sdkMcpServers`: SDK MCP 服务器列表
   - `jsonSchema`: JSON Schema 定义
   - `systemPrompt`: 系统提示
   - `appendSystemPrompt`: 追加系统提示
   - `agents`: 工作流定义
   - `promptSuggestions`: 提示建议开关
   - `agentProgressSummaries`: 工作流进度摘要开关

2. **SDKControlInitializeResponseSchema** — 初始化响应
   - `commands`: 斜杠命令列表
   - `agents`: 工作流信息列表
   - `output_style`: 输出样式
   - `available_output_styles`: 可用输出样式
   - `models`: 模型信息
   - `account`: 账户信息
   - `pid`: CLI 进程 PID
   - `fast_mode_state`: 快速模式状态

3. **SDKControlInterruptRequestSchema** — 中断请求

4. **SDKControlPermissionRequestSchema** — 权限请求
   - 工具使用权限检查
   - 包含工具名、输入、建议等

5. **SDKControlSetPermissionModeRequestSchema** — 设置权限模式

6. **SDKControlSetModelRequestSchema** — 设置模型

7. **SDKControlSetMaxThinkingTokensRequestSchema** — 设置最大思考 tokens

8. **SDKControlMcpStatusRequestSchema/ResponseSchema** — MCP 状态查询

9. **SDKControlGetContextUsageRequestSchema/ResponseSchema** — 上下文使用统计
   - 类别统计
   - Grid 布局
   - 模型信息
   - 内存文件
   - MCP 工具
   - API 使用统计

10. **SDKControlRewindFilesRequestSchema/ResponseSchema** — 文件回溯

11. **SDKControlCancelAsyncMessageRequestSchema/ResponseSchema** — 取消异步消息

12. **SDKControlSeedReadStateRequestSchema** — 种子读取状态

13. **SDKControlMcpMessageRequestSchema** — MCP 消息发送

14. **SDKControlMcpSetServersRequestSchema/ResponseSchema** — 设置 MCP 服务器

15. **SDKControlReloadPluginsRequestSchema/ResponseSchema** — 重新加载插件

16. **SDKControlMcpReconnectRequestSchema** — MCP 重连

17. **SDKControlMcpToggleRequestSchema** — MCP 开关

18. **SDKControlStopTaskRequestSchema** — 停止任务

19. **SDKControlApplyFlagSettingsRequestSchema** — 应用标志设置

20. **SDKControlGetSettingsRequestSchema/ResponseSchema** — 获取设置

21. **SDKControlElicitationRequestSchema/ResponseSchema** — 请求用户输入

### 请求/响应包装

**SDKControlRequestSchema**:
- `type`: 'control_request'
- `request_id`: 请求 ID
- `request`: 内部请求联合类型

**SDKControlResponseSchema**:
- `type`: 'control_response'
- `response`: 成功或错误响应

**ControlResponseSchema**:
- `subtype`: 'success'
- `request_id`: 请求 ID
- `response`: 响应数据

**ControlErrorResponseSchema**:
- `subtype`: 'error'
- `request_id`: 请求 ID
- `error`: 错误消息
- `pending_permission_requests`: 待处理权限请求

**SDKControlCancelRequestSchema** — 取消控制请求

**SDKKeepAliveMessageSchema** — 保持连接消息

**SDKUpdateEnvironmentVariablesMessageSchema** — 更新环境变量

### 聚合消息类型

**StdoutMessageSchema** — 标准输出消息联合
**StdinMessageSchema** — 标准输入消息联合

## 设计要点

1. **延迟 Schema**:
   - 使用 `lazySchema` 包装所有 schema
   - 避免循环依赖
   - 支持运行时创建

2. **联合类型**:
   - 请求和响应使用 z.union
   - 支持多种子类型

3. **描述性文档**:
   - 每个 schema 都有详细描述
   - 明确说明用途和行为

4. **可选字段**:
   - 使用 `.optional()` 标记可选
   - 合理的默认值

5. **内部标记**:
   - `@internal` 标记内部使用字段
   - `ultraplan` 等字段标记

## 与其他文件的关系

- **./coreSchemas.js**: 导入基础 schema
- **../../utils/lazySchema.js**: lazySchema 实现
- **SDK builders**: 使用这些 schema 进行通信

## 注意事项

1. 这些 schema 用于 SDK 与 CLI 之间的控制协议
2. SDK 消费者应使用 coreSchemas.ts
3. 所有 schema 都是延迟创建的
4. 响应包含成功和错误两种子类型
5. 支持取消正在进行的请求
6. 保持连接消息用于 WebSocket 连接维护
