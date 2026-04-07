# metadata.ts — 分析事件元数据管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/analytics/metadata.ts`
- **所属模块**: Analytics Service
- **功能类型**: 元数据收集与格式化

## 功能概述

该模块提供跨所有分析系统（Datadog、1P）共享的事件元数据收集和格式化功能。它是元数据的单一真相源，负责收集环境、运行时和上下文信息。

## 核心内容详解

### 工具名称清理

#### `sanitizeToolNameForAnalytics(toolName): string`
清理工具名称以保护 PII。
- MCP 工具（`mcp__*`）返回 `"mcp_tool"`
- 内置工具（Bash、Read 等）保留原名

#### `isToolDetailsLoggingEnabled(): boolean`
检查是否启用了详细工具名称日志记录（通过 `OTEL_LOG_TOOL_DETAILS` 环境变量）。

#### `isAnalyticsToolDetailsLoggingEnabled(mcpServerType, mcpServerBaseUrl): boolean`
检查 MCP 服务器/工具名称的详细日志记录是否启用：
- Cowork（entrypoint=local-agent）— 无 ZDR 概念，记录所有 MCP
- claude.ai 代理连接器 — 始终为官方
- 官方 MCP 注册表匹配的 URL
- 自定义/用户配置的 MCP 保持清理状态

#### `extractMcpToolDetails(toolName): { serverName, mcpToolName } | undefined`
从完整 MCP 工具名提取服务器和工具名。
格式：`mcp__<server>__<tool>`

### 文件扩展名提取

#### `getFileExtensionForAnalytics(filePath): string | undefined`
提取并清理文件扩展名用于分析日志记录。
- 使用 Node 的 `path.extname`
- 超过 10 字符的扩展名替换为 `"other"`（避免记录哈希文件名）

#### `getFileExtensionsFromBashCommand(command, simulatedSedEditFilePath): string | undefined`
从 Bash 命令提取文件扩展名。
支持的命令：`rm`, `mv`, `cp`, `touch`, `mkdir`, `chmod`, `chown`, `cat`, `head`, `tail`, `sort`, `stat`, `diff`, `wc`, `grep`, `rg`, `sed`

### 工具输入序列化

#### `truncateToolInputValue(value, depth): unknown`
截断工具输入值以控制输出大小。
- 字符串超过 512 字符截断到 128 字符
- 最大集合项数：20
- 最大深度：2
- 过滤内部标记键（`_` 开头）

#### `extractToolInputForTelemetry(input): string | undefined`
序列化工具输入参数用于 OTel `tool_result` 事件。
- 需要 `OTEL_LOG_TOOL_DETAILS` 启用
- 最大 JSON 字符数：4096
- 保留文件路径、URL 和 MCP 参数等法医有用字段

### 元数据类型

#### `EnvContext`
环境上下文元数据，包括：
- 平台信息（platform, arch, nodeVersion）
- 终端和包管理器
- CI/CD 检测（isCi, isGithubAction, isClaudeCodeAction）
- 远程环境（isClaudeCodeRemote, remoteEnvironmentType）
- 版本信息（version, versionBase, buildTime）
- WSL 信息（wslVersion, linuxDistroId 等）
- VCS 检测

#### `ProcessMetrics`
进程指标，包括：
- 运行时间（uptime）
- 内存使用（rss, heapTotal, heapUsed, external, arrayBuffers）
- 受限内存（constrainedMemory）
- CPU 使用率和百分比

#### `EventMetadata`
核心事件元数据，包括：
- 模型和会话信息
- 用户类型和 Beta 功能
- 环境上下文
- 入口点和 Agent SDK 版本
- 交互状态和客户端类型
- 进程指标
- SWE-bench 标识符
- Swarm/团队 Agent 识别（agentId, parentSessionId, agentType, teamName）
- 订阅类型和仓库哈希
- Kairos 和技能模式

### 主要函数

#### `getEventMetadata(options): Promise<EventMetadata>`
获取所有分析系统共享的核心事件元数据。

**收集流程：**
1. 获取模型和 Beta 配置
2. 并行构建环境上下文和仓库远程哈希
3. 构建进程指标
4. 组合完整元数据对象

#### `to1PEventFormat(metadata, userMetadata, additionalMetadata): FirstPartyEventLoggingMetadata`
将元数据转换为 1P 事件日志格式（snake_case 字段）。

**转换内容：**
- `envContext` → `env`（EnvironmentMetadata）
- `processMetrics` → `process`（base64）
- `userMetadata` → `auth`
- 核心字段 → `core`（snake_case）
- 额外元数据 → `additional`（base64 JSON）

### Agent 识别

#### `getAgentIdentification(): { agentId?, parentSessionId?, agentType?, teamName? }`
获取 Agent 识别信息。

**优先级：**
1. AsyncLocalStorage 上下文（子 Agent）
2. 环境变量（Swarm 队友）
3. Bootstrap 状态（父会话 ID）

## 设计要点

1. **单一真相源** — 所有分析系统共享相同的元数据收集逻辑
2. **PII 保护** — 工具名称和扩展名清理，避免暴露用户配置
3. **性能优化** — 使用 memoize 缓存环境上下文
4. **类型安全** — 完整 TypeScript 类型定义，proto 生成的类型
5. **Agent 感知** — 支持 Swarm、子 Agent 和独立 Agent 的识别

## 与其他文件的关系

- **被调用**: `datadog.ts`, `firstPartyEventLogger.ts`
- **依赖**: `../../utils/` 中的各种工具函数
- **关联**: `officialRegistry.ts` 检查官方 MCP URL

## 注意事项

- 添加新字段到 `env` 需要先更新 monorepo proto
- 环境上下文使用 memoize 缓存，但 `kairosActive` 在外部读取（避免 memoization 问题）
- 进程指标包括 CPU% 差值跟踪（进程全局状态）
- 版本基础从完整版本提取（例如：2.0.36-dev.20251107... → 2.0.36-dev）
