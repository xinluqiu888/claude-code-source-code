# bridgeConfig.ts — Bridge API 认证与 URL 配置统一入口

> **一句话总结**：集中管理 bridge API 的 OAuth Token 获取和 Base URL 解析，支持开发者环境变量覆盖与生产 OAuth 配置的两层回退逻辑。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/bridge/bridgeConfig.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 49 |
| 主要职责 | 提供统一的 bridge API 访问 Token 和 Base URL 获取函数，消除跨文件的重复 `CLAUDE_BRIDGE_*` 环境变量读取 |

---

## 功能概述

`bridgeConfig.ts` 解决了此前在十余个文件中重复存在的 Anthropic 内部开发者环境变量（`CLAUDE_BRIDGE_OAUTH_TOKEN`、`CLAUDE_BRIDGE_BASE_URL`）读取逻辑，将其整合为四个简单函数。

文件的设计采用"两层"模式：`*Override()` 函数返回 Anthropic 内部（`USER_TYPE === 'ant'`）的环境变量值（或 `undefined`）；非 Override 版本则将 Override 结果与真实 OAuth Token/配置进行 `??` 回退。这样，Daemon 调用方（使用 IPC 认证而非 OAuth）可以直接使用 `getBridgeTokenOverride()` 而不影响主流程。

---

## 核心内容详解

### 导入与依赖

| 依赖 | 用途 |
|------|------|
| `../constants/oauth.js` | `getOauthConfig()` — 获取生产 OAuth 配置（含 `BASE_API_URL`） |
| `../utils/auth.js` | `getClaudeAIOAuthTokens()` — 从钥匙串/缓存获取 claude.ai OAuth Token |

### 主要函数

#### `getBridgeTokenOverride(): string | undefined`
仅在 `USER_TYPE === 'ant'` 时返回 `CLAUDE_BRIDGE_OAUTH_TOKEN` 环境变量值，否则返回 `undefined`。

#### `getBridgeBaseUrlOverride(): string | undefined`
仅在 `USER_TYPE === 'ant'` 时返回 `CLAUDE_BRIDGE_BASE_URL` 环境变量值，否则返回 `undefined`。

#### `getBridgeAccessToken(): string | undefined`
优先使用 `getBridgeTokenOverride()`，若无则读取 `getClaudeAIOAuthTokens()?.accessToken`。返回 `undefined` 表示未登录。

#### `getBridgeBaseUrl(): string`
优先使用 `getBridgeBaseUrlOverride()`，若无则使用 `getOauthConfig().BASE_API_URL`（生产地址）。**始终返回一个 URL**（无 `undefined`）。

### 数据流与逻辑流程

```
getBridgeAccessToken()
  → getBridgeTokenOverride() → process.env.CLAUDE_BRIDGE_OAUTH_TOKEN (ant-only)
  → getClaudeAIOAuthTokens()?.accessToken (keychain / cache)

getBridgeBaseUrl()
  → getBridgeBaseUrlOverride() → process.env.CLAUDE_BRIDGE_BASE_URL (ant-only)
  → getOauthConfig().BASE_API_URL (production)
```

### 对外接口（导出内容）

| 导出 | 类型 | 说明 |
|------|------|------|
| `getBridgeTokenOverride` | 函数 | Ant-only Token 覆盖值 |
| `getBridgeBaseUrlOverride` | 函数 | Ant-only Base URL 覆盖值 |
| `getBridgeAccessToken` | 函数 | 最终生效的 Bridge OAuth Token |
| `getBridgeBaseUrl` | 函数 | 最终生效的 Bridge API Base URL |

---

## 设计要点

- **零重复原则**：消除了此前散落在 `inboundAttachments`、`bridgeMain`、`initReplBridge`、`remoteBridgeCore` 等文件中的重复环境变量读取代码。
- **`ant-only` 安全隔离**：`USER_TYPE === 'ant'` 检查确保内部开发者覆盖配置不会泄漏到外部构建。
- **接口分层**：Override getter 供特殊调用方直接使用（如 Daemon 使用 IPC 认证时需要独立的 Override 而不需要整体 `getBridgeAccessToken` 逻辑）。

---

## 与其他文件的关系

- **依赖**：
  - `../constants/oauth.js`
  - `../utils/auth.js`
- **被依赖**：
  - `inboundAttachments.ts`：下载附件时获取 Token 和 URL
  - `bridgeMain.ts`：（间接，通过其他模块）
  - `initReplBridge.ts`：读取 Token 和 URL
  - `remoteBridgeCore.ts`：env-less bridge 核心

---

## 注意事项

- `getBridgeBaseUrl()` 永远不会返回 `undefined`（与 `getBridgeAccessToken()` 的可选性不同），调用方可安全使用。
- Daemon/CLI 模式下若使用环境变量 Token（非 OAuth），不应使用 `getBridgeAccessToken()`，而应直接使用 `getBridgeTokenOverride()` 或由注入的 `getAccessToken` 提供。
