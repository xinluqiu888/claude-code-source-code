# auth.ts — MCP 认证处理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/mcp/auth.ts`
- **所属模块**: MCP Service
- **功能类型**: MCP OAuth 认证

## 功能概述

处理 MCP 服务器的 OAuth 认证流程，支持 Claude AI 作为认证提供者。

## 核心内容详解

### ClaudeAuthProvider

实现 MCP SDK 的 `OAuthClientProvider` 接口。

**功能：**
- `redirectUrl` — 重定向 URL
- `clientMetadata` — 客户端元数据
- `codeChallenge` / `codeChallengeMethod` — PKCE
- `authCode` — 授权码获取

### 关键函数

#### `hasMcpDiscoveryButNoToken(serverName): Promise<boolean>`
检查服务器支持 OAuth 但无令牌。

**流程：**
1. 读取 OAuth 配置
2. 检查 discovery 信息
3. 验证令牌存在性

#### `wrapFetchWithStepUpDetection(fetch, serverName)`
包装 fetch 检测 step-up 认证需求。

**行为：**
- 401 时触发重新认证
- 记录分析事件

### OAuth 流程

1. 发现服务器 OAuth 配置
2. 生成 PKCE 参数
3. 打开浏览器授权
4. 交换授权码
5. 存储令牌

## 设计要点

1. **PKCE 支持** — 安全 OAuth 流程
2. **令牌管理** — 安全存储和刷新
3. **Step-up 检测** — 动态认证需求
4. **错误处理** — 401 触发重新认证

## 与其他文件的关系

- **实现**: `@modelcontextprotocol/sdk/auth`
- **被调用**: `client.ts`, `claudeai.ts`

## 注意事项

- OAuth 令牌存储在系统密钥链
- 支持自动令牌刷新
- Step-up 认证可能随时触发
