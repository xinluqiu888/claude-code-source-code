# logging.ts — API 日志记录服务

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/api/logging.ts`
- **所属模块**: API Service
- **功能类型**: API 请求/响应日志记录

## 功能概述

该模块提供 API 请求和响应的详细日志记录功能，包括查询日志、错误日志和成功日志。支持遥测事件、分析事件和调试日志的多重输出。

## 核心内容详解

### 常量与类型

#### `EMPTY_USAGE`
空使用数据占位符，用于无法获取使用统计时。

#### `GlobalCacheStrategy`
全局缓存策略类型：
- `'tool_based'` — 基于工具的缓存
- `'system_prompt'` — 系统提示词缓存
- `'none'` — 无全局缓存

### 网关检测

#### `KnownGateway`
已知的 AI 网关类型：
- `litellm` — LiteLLM 代理
- `helicone` — Helicone
- `portkey` — Portkey
- `cloudflare-ai-gateway` — Cloudflare AI Gateway
- `kong` — Kong AI Gateway
- `braintrust` — Braintrust
- `databricks` — Databricks AI Gateway

#### `detectGateway({ headers, baseUrl }): KnownGateway | undefined`
从响应头或基础 URL 检测使用的 AI 网关。

**检测方法：**
- 响应头前缀匹配（如 `x-litellm-`、`helicone-`）
- 主机名后缀匹配（如 `.cloud.databricks.com`）

### 日志函数

#### `logAPIQuery(options): void`
记录 API 查询请求。

**记录字段：**
- 模型、消息数量、温度
- Beta 功能列表
- 权限模式
- 查询来源和跟踪信息
- 思考类型和努力值
- Fast 模式状态
- 构建年龄
- 提供商
- 环境元数据（ANTHROPIC_BASE_URL 等）

**事件名称：** `tengu_api_query`

#### `logAPIError(options): void`
记录 API 错误。

**记录字段：**
- 错误信息和类型
- 状态码
- 模型和消息统计
- 持续时间（含重试）
- 尝试次数
- 网关检测
- 请求/客户端 ID
- 连接错误详情
- 查询跟踪信息
- Fast 模式状态

**输出目标：**
- 分析事件：`tengu_api_error`
- OTLP 事件：`api_error`
- LLM 追踪 span（结束标记）
- 调试日志（ant 用户）
- Teleport 首次消息错误（如果是 teleport 会话）

**事件名称：**
- `tengu_api_error` — 主要分析事件
- `tengu_teleport_first_message_error` — Teleport 会话首次消息错误

#### `logAPISuccessAndDuration(options): void`
记录 API 成功响应。

**流程：**
1. 检测网关
2. 计算内容长度统计
3. 计算持续时间
4. 调用 `logAPISuccess`
5. 记录 OTLP 事件
6. 结束 LLM 追踪 span
7. 检查 Teleport 会话首次消息

**内容统计：**
- `textContentLength` — 文本内容长度
- `thinkingContentLength` — 思考内容长度（ant 用户）
- `toolUseContentLengths` — 各工具使用长度
- `connectorTextBlockCount` — 连接器文本块数量

#### `logAPISuccess(options): void`
核心成功日志记录（内部使用）。

**记录字段：**
- 输入/输出 Token 数
- 缓存读/写 Token 数
- 首次 Token 时间（TTFT）
- 停止原因
- 成本（USD）
- 是否非交互式会话
- 查询链信息
- 全局缓存策略
- 自上次 API 调用的时间

**事件名称：** `tengu_api_success`

### 辅助函数

#### `getAnthropicEnvMetadata()`
获取 Anthropic 环境元数据（baseUrl、envModel、envSmallFastModel）。

#### `getBuildAgeMinutes(): number | undefined`
计算构建年龄（分钟）。

#### `getErrorMessage(error): string`
从 APIError 提取错误消息。

## 设计要点

1. **多目标输出** — 分析事件、OTLP 遥测、LLM 追踪
2. **网关感知** — 自动检测并记录 AI 网关使用
3. **丰富元数据** — Token 统计、成本、性能指标
4. **Teleport 跟踪** — 专门处理 teleport 会话的首次消息
5. **内容分析** — 文本、思考、工具使用的长度统计

## 与其他文件的关系

- **被调用**: `claude.ts` 的 API 调用流程
- **依赖**: `../analytics/index.ts` 分析事件
- **关联**: `../../utils/telemetry/sessionTracing.ts` LLM 追踪

## 注意事项

- 仅当 `newMessages` 提供时计算内容长度
- 思考内容仅对 ant 用户记录
- 成本计算使用 `calculateUSDCost`
- LLM span 需要正确匹配请求和响应
