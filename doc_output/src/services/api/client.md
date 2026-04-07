# client.ts — Anthropic API 客户端创建

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/api/client.ts`
- **所属模块**: API Service
- **功能类型**: API 客户端配置

## 功能概述

该模块负责创建和配置 Anthropic API 客户端，支持多种部署模式：直接 API、AWS Bedrock、Microsoft Foundry (Azure) 和 Google Vertex AI。根据环境变量自动选择正确的客户端实现。

## 核心内容详解

### 支持的环境变量

#### 直接 API 认证
- `ANTHROPIC_API_KEY` — API 密钥（直接 API 访问必需）

#### AWS Bedrock
- `AWS_REGION` / `AWS_DEFAULT_REGION` — AWS 区域（默认：us-east-1）
- `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` — Haiku 模型的区域覆盖
- `CLAUDE_CODE_USE_BEDROCK` — 启用 Bedrock 模式
- `CLAUDE_CODE_SKIP_BEDROCK_AUTH` — 跳过认证（测试用）
- `AWS_BEARER_TOKEN_BEDROCK` — Bedrock API 密钥认证

#### Microsoft Foundry (Azure)
- `ANTHROPIC_FOUNDRY_RESOURCE` — Azure 资源名称
- `ANTHROPIC_FOUNDRY_BASE_URL` — 完整的 Base URL
- `ANTHROPIC_FOUNDRY_API_KEY` — Foundry API 密钥
- `CLAUDE_CODE_USE_FOUNDRY` — 启用 Foundry 模式
- `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` — 跳过认证（测试用）

#### Vertex AI
- `VERTEX_REGION_*` — 各模型的区域配置
- `CLOUD_ML_REGION` — 默认 GCP 区域
- `ANTHROPIC_VERTEX_PROJECT_ID` — GCP 项目 ID
- `CLAUDE_CODE_USE_VERTEX` — 启用 Vertex 模式
- `CLAUDE_CODE_SKIP_VERTEX_AUTH` — 跳过认证（测试用）

### 主要函数

#### `getAnthropicClient(options): Promise<Anthropic>`
创建适当配置的 Anthropic 客户端。

**参数：**
- `apiKey` — 可选 API 密钥
- `maxRetries` — 最大重试次数
- `model` — 模型名称（用于区域选择）
- `fetchOverride` — 自定义 fetch 实现
- `source` — 调用来源标识

**客户端选择逻辑：**
1. 检查 `CLAUDE_CODE_USE_BEDROCK` — 使用 `AnthropicBedrock`
2. 检查 `CLAUDE_CODE_USE_FOUNDRY` — 使用 `AnthropicFoundry`
3. 检查 `CLAUDE_CODE_USE_VERTEX` — 使用 `AnthropicVertex`
4. 默认 — 使用标准 `Anthropic` 客户端

**标准客户端配置：**
- 默认请求头：`x-app: cli`, `User-Agent`, `X-Claude-Code-Session-Id`
- 容器/远程会话 ID 头（如适用）
- 客户端应用头（`CLAUDE_AGENT_SDK_CLIENT_APP`）
- 额外保护头（`CLAUDE_CODE_ADDITIONAL_PROTECTION`）
- OAuth 令牌刷新检查
- 认证头配置

### 辅助函数

#### `configureApiKeyHeaders(headers, isNonInteractiveSession): Promise<void>`
配置 API 密钥认证头。
- 优先使用 `ANTHROPIC_AUTH_TOKEN`
- 回退到 apiKeyHelper

#### `getCustomHeaders(): Record<string, string>`
解析 `ANTHROPIC_CUSTOM_HEADERS` 环境变量。
- 支持多行格式（curl 风格）
- 格式：`Name: Value`

#### `buildFetch(fetchOverride, source): ClientOptions['fetch']`
构建自定义 fetch 包装器。
- 注入 `x-client-request-id` 头（仅限 First Party API）
- 记录 API 请求路径和来源
- 用于超时调试和服务器日志关联

### 常量

#### `CLIENT_REQUEST_ID_HEADER = 'x-client-request-id'`
客户端生成的请求 ID 头名称。用于：
- 超时情况下的服务器日志关联
- 调试和故障排查

## 设计要点

1. **多提供商支持** — 统一接口支持 First Party、Bedrock、Foundry、Vertex
2. **环境驱动** — 通过环境变量自动选择客户端类型
3. **认证灵活** — 支持 API 密钥、OAuth、AWS/GCP 凭证
4. **调试友好** — 详细的调试日志和请求追踪
5. **容错处理** — 认证失败时的优雅降级

## 与其他文件的关系

- **被调用**: `claude.ts` 的 API 调用逻辑
- **依赖**: `../../utils/auth.ts` 获取认证信息
- **关联**: `../../utils/proxy.ts` 获取代理配置

## 注意事项

- OAuth 令牌在创建客户端前会自动刷新（如果需要）
- 第三方客户端返回类型被强制转换为 `Anthropic`（实际 API 支持可能不同）
- Vertex 认证使用 `google-auth-library`，支持多种凭证来源
- Foundry 使用 Azure AD 认证（DefaultAzureCredential）
