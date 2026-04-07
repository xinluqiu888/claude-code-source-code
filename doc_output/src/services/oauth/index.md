# index.ts — OAuth 服务入口

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/oauth/index.ts`
- **所属模块**: OAuth Service
- **功能类型**: OAuth 流程编排

## 功能概述

OAuth 2.0 授权码流程（含 PKCE）的主入口，支持自动和手动两种授权码获取方式。

## 核心内容详解

### OAuthService 类

#### 构造函数
- 生成 `codeVerifier`（PKCE）

#### `startOAuthFlow(authURLHandler, options)`
启动 OAuth 流程。

**选项：**
- `loginWithClaudeAi` — 使用 Claude.ai 登录
- `inferenceOnly` — 仅推理范围
- `expiresIn` — 令牌有效期
- `orgUUID` — 组织 UUID
- `loginHint` — 登录提示（预填充邮箱）
- `loginMethod` — 登录方法（sso/magic_link/google）
- `skipBrowserOpen` — 跳过浏览器打开（SDK 用）

**流程：**
1. 启动本地回调监听器（`AuthCodeListener`）
2. 生成 PKCE challenge 和 state
3. 构建自动和手动两种授权 URL
4. 等待授权码（自动回调或手动输入）
5. 交换令牌
6. 获取用户信息
7. 处理成功重定向（自动流）

#### `handleManualAuthCodeInput(params)`
处理手动输入的授权码。

#### `cleanup()`
清理资源（关闭监听器）。

### 令牌格式化

```typescript
{
  accessToken: string
  refreshToken: string
  expiresAt: number
  scopes: string[]
  subscriptionType: SubscriptionType | null
  rateLimitTier: RateLimitTier | null
  profile: OAuthProfileResponse
  tokenAccount: { uuid, emailAddress, organizationUuid }
}
```

## 设计要点

1. **双模式支持** — 自动（浏览器回调）和手动（复制粘贴）
2. **PKCE 安全** — 防止授权码拦截攻击
3. **灵活选项** — 支持多种登录场景
4. **资源清理** — 确保监听器关闭

## 与其他文件的关系

- **依赖**: `auth-code-listener.ts`, `client.ts`, `crypto.ts`
- **被调用**: `auth.ts` 登录流程

## 注意事项

- 自动流需要浏览器支持 localhost 回调
- 手动流用于 SSH/容器等环境
- 令牌有效期可选（默认由服务器决定）
