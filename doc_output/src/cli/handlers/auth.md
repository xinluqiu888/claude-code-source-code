# handlers/auth.ts — 认证子命令处理器

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/handlers/auth.ts` |
| **文件类型** | TypeScript 处理器模块 |
| **行数** | 331 行 |
| **职责** | 实现 `claude auth` 子命令（login、logout、status），处理 OAuth 认证流程 |

## 功能概述

本模块实现 Claude Code 的认证子命令处理器，提供完整的登录、登出和状态查看功能。支持多种认证方式：

- **OAuth 登录**：通过浏览器或环境变量刷新令牌
- **多登录方式**：支持 claude.ai 和 Console 两种登录方式
- **SSO 登录**：支持单点登录
- **状态查看**：以 JSON 或文本格式显示当前认证状态
- **登出**：清除所有认证相关状态

模块处理完整的 OAuth 流程，包括令牌获取、用户信息存储、API Key 创建（Console 用户）、缓存清理等。

## 核心内容详解

### 导入

| 导入项 | 来源 | 用途 |
|--------|------|------|
| `performLogout`, `clearAuthRelatedCaches` | `../../commands/logout/logout.js` | 登出和缓存清理 |
| `logEvent` | `../../services/analytics` | 分析事件记录 |
| `getSSLErrorHint` | `../../services/api/errorUtils.js` | SSL 错误提示 |
| `fetchAndStoreClaudeCodeFirstTokenDate` | `../../services/api/firstTokenDate.js` | 记录首次使用日期 |
| `createAndStoreApiKey`, `fetchAndStoreUserRoles`, `refreshOAuthToken`, `shouldUseClaudeAIAuth`, `storeOAuthAccountInfo` | `../../services/oauth/client.js` | OAuth 客户端功能 |
| `getOauthProfileFromOauthToken` | `../../services/oauth/getOauthProfile.js` | 获取用户资料 |
| `OAuthService` | `../../services/oauth/index.js` | OAuth 服务 |
| `OAuthTokens` | `../../services/oauth/types.js` | OAuth 令牌类型 |
| `getAnthropicApiKeyWithSource`, `getAuthTokenSource`, `getOauthAccountInfo`, `getSubscriptionType`, `isUsing3PServices`, `saveOAuthTokensIfNeeded`, `validateForceLoginOrg` | `../../utils/auth.js` | 认证工具函数 |
| `saveGlobalConfig` | `../../utils/config.js` | 全局配置保存 |
| `logForDebugging` | `../../utils/debug.js` | 调试日志 |
| `isRunningOnHomespace` | `../../utils/envUtils.js` | 环境检测 |
| `errorMessage` | `../../utils/errors.js` | 错误信息提取 |
| `logError` | `../../utils/log.js` | 错误日志 |
| `getAPIProvider` | `../../utils/model/providers.js` | API 提供商获取 |
| `getInitialSettings` | `../../utils/settings/settings.js` | 初始设置 |
| `jsonStringify` | `../../utils/slowOperations.js` | JSON 序列化 |
| `buildAccountProperties`, `buildAPIProviderProperties` | `../../utils/status.js` | 状态属性构建 |

### 核心函数

#### `export async function installOAuthTokens(tokens: OAuthTokens): Promise<void>`

共享的令牌获取后处理逻辑。保存令牌、获取用户资料/角色、设置本地认证状态。

**执行流程**：

1. **清除旧状态**
   ```typescript
   await performLogout({ clearOnboarding: false })
   ```
   在保存新凭证前清除旧状态，避免冲突。

2. **获取用户资料**
   - 优先使用预获取的资料（令牌中包含）
   - 否则通过 access token 获取
   - 存储账户信息：UUID、邮箱、组织 UUID、显示名等

3. **保存令牌**
   ```typescript
   const storageResult = saveOAuthTokensIfNeeded(tokens)
   ```
   保存 OAuth 令牌，清除令牌缓存。

4. **获取角色和首次使用日期**
   - 获取用户角色（可能失败，对有限范围令牌）
   - 对于 claude.ai 认证，记录首次使用日期

5. **创建 API Key（Console 用户）**
   ```typescript
   const apiKey = await createAndStoreApiKey(tokens.accessToken)
   ```
   对 Console 用户创建 API Key，这是关键步骤，失败会抛出错误。

6. **清理缓存**
   ```typescript
   await clearAuthRelatedCaches()
   ```

#### `export async function authLogin(options)`

处理 `claude auth login` 命令。

**参数**：
- `email`: 可选，登录提示邮箱
- `sso`: 可选，使用 SSO 登录
- `console`: 可选，使用 Console 登录（而非 claude.ai）
- `claudeai`: 可选，使用 claude.ai 登录

**互斥检查**：
```typescript
if (useConsole && claudeai) {
  process.stderr.write('Error: --console and --claudeai cannot be used together.\n')
  process.exit(1)
}
```

**登录方式解析**：
- 如果设置了 `forceLoginMethod`，强制使用指定方式
- 否则 `--console` 选择 Console，否则选择 claude.ai

**环境变量快速路径**：
如果设置了 `CLAUDE_CODE_OAUTH_REFRESH_TOKEN`：
1. 检查 `CLAUDE_CODE_OAUTH_SCOPES` 是否设置
2. 直接交换刷新令牌获取访问令牌
3. 跳过浏览器流程

**标准 OAuth 流程**：
1. 创建 `OAuthService` 实例
2. 启动 OAuth 流（打开浏览器）
3. 获取令牌后调用 `installOAuthTokens`
4. 验证强制登录组织（如果配置）
5. 标记 onboarding 完成
6. 显示成功消息并退出

**错误处理**：
- 记录错误日志
- 显示 SSL 错误提示（如果适用）
- 显示失败消息并退出（错误码 1）

#### `export async function authStatus(opts)`

处理 `claude auth status` 命令，显示当前认证状态。

**输出格式**：

**JSON 格式**（默认）：
```json
{
  "loggedIn": true,
  "authMethod": "claude.ai",
  "apiProvider": "anthropic",
  "apiKeySource": "/login managed key",
  "email": "user@example.com",
  "orgId": "org-uuid",
  "orgName": "Org Name",
  "subscriptionType": "team"
}
```

**文本格式**（`--text`）：
显示人类可读的属性列表。

**认证方法检测逻辑**：
| 条件 | authMethod |
|------|------------|
| 使用第三方服务 | `third_party` |
| claude.ai 令牌源 | `claude.ai` |
| API Key 辅助 | `api_key_helper` |
| 其他 OAuth 令牌 | `oauth_token` |
| ANTHROPIC_API_KEY 环境变量 | `api_key` |
| /login 管理的 key | `claude.ai` |

**退出码**：
- 已登录：0
- 未登录：1

#### `export async function authLogout()`

处理 `claude auth logout` 命令。

**执行流程**：
1. 调用 `performLogout({ clearOnboarding: false })`
2. 成功：显示成功消息，退出码 0
3. 失败：显示错误消息，退出码 1

## 设计要点

1. **统一的令牌安装**：`installOAuthTokens` 函数统一处理所有登录成功后的逻辑，确保状态一致性
2. **环境变量支持**：支持通过 `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` 进行无浏览器登录，便于 CI/CD
3. **组织强制验证**：支持通过配置强制特定组织登录，用于企业场景
4. **多格式输出**：认证状态支持 JSON 和文本两种输出格式
5. **错误处理**：详细的错误信息，包括 SSL 错误提示
6. **分析集成**：记录登录开始、成功等事件用于产品分析

## 与其他文件的关系

| 关系类型 | 文件 | 描述 |
|---------|------|------|
| **导入** | `src/services/oauth/index.js` | OAuthService 主类 |
| **导入** | `src/services/oauth/client.js` | OAuth 客户端功能 |
| **导入** | `src/services/analytics` | 分析事件记录 |
| **导入** | `src/utils/auth.js` | 认证工具函数 |
| **导入** | `src/utils/config.js` | 全局配置 |
| **导入** | `src/commands/logout/logout.js` | 登出功能 |
| **被导入** | `src/main.tsx` | `claude auth` 命令处理 |
| **被导入** | `src/cli/print.ts` | 安装 OAuth 令牌 |

## 注意事项

1. **process.exit 使用**：模块顶部注释 `eslint-disable custom-rules/no-process-exit`，因为这是 CLI 子命令处理器，需要显式控制退出
2. **令牌范围要求**：使用环境变量登录时，`CLAUDE_CODE_OAUTH_SCOPES` 是必需的，用于指定刷新令牌的权限范围
3. **API Key 关键性**：Console 用户的 API Key 创建是关键步骤，失败会导致登录失败
4. **组织验证**：`validateForceLoginOrg()` 可能拒绝登录，如果用户不属于强制要求的组织
5. **状态缓存**：登出时不清除 `hasCompletedOnboarding` 配置，保持用户体验连贯性
6. **错误信息**：使用 `errorMessage()` 提取友好错误信息，`getSSLErrorHint()` 提供 SSL 相关问题的额外提示
