# client.ts — OAuth 客户端

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/oauth/client.ts`
- **所属模块**: OAuth Service
- **功能类型**: OAuth HTTP API 客户端

## 功能概述

处理 OAuth HTTP 请求，包括授权 URL 构建、令牌交换、刷新和用户信息获取。

## 核心内容详解

### 主要函数

#### `shouldUseClaudeAIAuth(scopes): boolean`
检查是否使用 Claude.ai 认证。
- 检查 `CLAUDE_AI_INFERENCE_SCOPE`

#### `parseScopes(scopeString): string[]`
解析空格分隔的范围字符串。

#### `buildAuthUrl(options): string`
构建授权 URL。

**参数：**
- `codeChallenge` — PKCE challenge
- `state` — CSRF 防护
- `port` — 本地回调端口
- `isManual` — 手动流程
- `loginWithClaudeAi` — Claude.ai 登录
- `inferenceOnly` — 仅推理范围
- `orgUUID` — 组织
- `loginHint` — 预填充邮箱
- `loginMethod` — 登录方法

#### `exchangeCodeForTokens(...): Promise<OAuthTokenExchangeResponse>`
交换授权码获取令牌。

**请求体：**
- `grant_type: 'authorization_code'`
- `code`, `code_verifier`, `state`
- `client_id`, `redirect_uri`
- `expires_in`（可选）

#### `refreshOAuthToken(refreshToken, options): Promise<OAuthTokens>`
刷新访问令牌。

**优化：**
- 已有完整资料时跳过 `/api/oauth/profile` 请求
- 减少每日约 700 万次请求

#### `fetchAndStoreUserRoles(accessToken): Promise<void>`
获取并存储用户角色。

## 设计要点

1. **URL 构建** — 支持多种授权场景
2. **令牌刷新优化** — 避免不必要的资料获取
3. **错误处理** — 401/403 清晰错误消息
4. **范围管理** — 支持动态范围请求

## 与其他文件的关系

- **依赖**: `../../constants/oauth.ts` 配置
- **被调用**: `index.ts`, `auth.ts`

## 注意事项

- 刷新令牌可能轮换（返回新令牌）
- 令牌交换 15 秒超时
- 分析事件记录成功/失败
