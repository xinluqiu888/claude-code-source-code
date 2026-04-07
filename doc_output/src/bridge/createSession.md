# createSession.ts — 桥接会话创建与管理 API

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/bridge/createSession.ts`
- **文件类型**: TypeScript 模块
- **行数**: 约 385 行
- **主要职责**: 提供桥接会话的创建、获取、归档和标题更新功能，支持 git 仓库上下文

## 功能概述

`createSession.ts` 实现了桥接会话的生命周期管理 API，包括会话创建、获取、归档和标题更新。这些功能供 `claude remote-control`（独立桥接）和 `/remote-control`（REPL 桥接）命令使用，用于在桥接环境中管理会话。

模块使用 CCR v2 Sessions API（`POST /v1/sessions` 等），通过 OAuth 认证和特定的 beta 头（`ccr-byoc-2025-07-29`）进行访问。与 `codeSessionApi.ts` 不同，本模块读取全局认证状态（`getClaudeAIOAuthTokens`），更适合 CLI 上下文使用。

会话创建支持预填充历史消息和 git 仓库上下文，使得用户可以在桥接环境中继续现有的对话。会话归档是显式操作，CCR 服务器不会自动归档会话。

## 核心内容详解

### 类型定义

#### `GitSource` 类型
```typescript
{
  type: 'git_repository'
  url: string
  revision?: string
}
```

#### `GitOutcome` 类型
```typescript
{
  type: 'git_repository'
  git_info: {
    type: 'github'
    repo: string
    branches: string[]
  }
}
```

#### `SessionEvent` 类型
```typescript
{
  type: 'event'
  data: SDKMessage
}
```

### createBridgeSession 函数

**功能**: 通过 POST `/v1/sessions` 创建桥接会话。

**参数**:
- `environmentId`: 环境 ID
- `title?`: 会话标题
- `events`: 预填充事件列表
- `gitRepoUrl`: Git 仓库 URL（可选）
- `branch`: 分支名
- `signal`: AbortSignal
- `baseUrl?`: API 基础 URL 覆盖
- `getAccessToken?`: 访问令牌获取函数
- `permissionMode?`: 权限模式

**Git 上下文构建**:
1. 解析 git 远程 URL（`parseGitRemote`）
2. 回退到 GitHub 仓库格式解析（`parseGitHubRepository`）
3. 获取默认分支（`getDefaultBranch`）
4. 构建 `GitSource` 和 `GitOutcome` 对象

**请求体**:
```typescript
{
  ...(title && { title }),
  events,
  session_context: {
    sources: gitSource ? [gitSource] : [],
    outcomes: gitOutcome ? [gitOutcome] : [],
    model: getMainLoopModel()
  },
  environment_id: environmentId,
  source: 'remote-control',
  ...(permissionMode && { permission_mode: permissionMode })
}
```

**认证头**:
- OAuth 头（`getOAuthHeaders`）
- `anthropic-beta`: `ccr-byoc-2025-07-29`
- `x-organization-uuid`: 组织 UUID

### getBridgeSession 函数

**功能**: 通过 GET `/v1/sessions/{id}` 获取会话信息。

**用途**: 用于 `--session-id` 恢复流程，获取会话的 `environment_id` 和标题。

**注意**: 使用 org-scoped 头（与 `bridgeApi.ts` 的环境级客户端不同），确保 Sessions API 返回正确结果。

### archiveBridgeSession 函数

**功能**: 通过 POST `/v1/sessions/{id}/archive` 归档会话。

**特性**:
- 接受任何状态的会话（running、idle、requires_action、pending）
- 已归档会话返回 409
- 安全调用（幂等）
- 调用者必须处理错误（函数内部无 try/catch）

**使用场景**:
- `claude remote-control` 关闭时
- REPL 桥接关闭时
- 显式用户操作

### updateBridgeSessionTitle 函数

**功能**: 通过 PATCH `/v1/sessions/{id}` 更新会话标题。

**使用场景**: 用户在桥接连接活跃时通过 `/rename` 重命名会话，同步到 claude.ai/code。

**会话 ID 兼容**: 使用 `toCompatSessionId` 将 `cse_*` 转换为 `session_*`，兼容 v2 和 v1 调用者。

**错误处理**: 错误被吞没，标题同步是最佳努力。

## 设计要点

1. **动态导入**: 使用 `await import()` 进行依赖导入，避免模块加载时的循环依赖和启动开销。

2. **Git 上下文智能解析**: 支持多种 git URL 格式（完整 URL、GitHub owner/repo），自动检测和回退。

3. **Org-Scoped 认证**: 使用组织 UUID 头，确保跨组织边界的正确授权。

4. **Beta 头版本**: 使用 `ccr-byoc-2025-07-29` beta 头访问 v2 Sessions API。

5. **会话 ID 兼容**: `updateBridgeSessionTitle` 内部转换会话 ID 格式，调用者无需关心版本差异。

6. **最佳努力归档**: 归档是清理操作，失败不应阻塞关机流程。

7. **错误透明**: 调试日志记录详细的失败原因（状态码、错误详情），便于问题排查。

8. **AbortSignal 支持**: `createBridgeSession` 接受 AbortSignal，支持请求取消。

## 与其他文件的关系

| 文件 | 关系描述 |
|------|----------|
| `../entrypoints/agentSdkTypes.js` | 导入 `SDKMessage` 类型 |
| `./debugUtils.js` | 导入 `extractErrorDetail` |
| `./sessionIdCompat.js` | 导入 `toCompatSessionId` 进行会话 ID 转换 |
| `../utils/auth.js` | 动态导入 `getClaudeAIOAuthTokens` |
| `../services/oauth/client.js` | 动态导入 `getOrganizationUUID` |
| `../constants/oauth.js` | 动态导入 `getOauthConfig` |
| `../utils/teleport/api.js` | 动态导入 `getOAuthHeaders` |
| `../utils/detectRepository.js` | 动态导入 git 解析函数 |
| `../utils/git.js` | 动态导入 `getDefaultBranch` |
| `../utils/model/model.js` | 动态导入 `getMainLoopModel` |
| `bridgeApi.ts` | 对比：使用环境级头，本模块使用 org-scoped 头 |

## 注意事项

1. **动态导入性能**: 每次调用都重新导入依赖模块，这在冷路径（归档、标题更新）是可接受的，但在热路径（会话创建）可能有轻微开销。

2. **Git 解析回退**: `parseGitRemote` 失败时尝试 `parseGitHubRepository`，后者期望 `owner/repo` 格式。

3. **分支名默认**: 如果未提供分支，使用 `getDefaultBranch()` 或默认到 `'task'`。

4. **归档幂等性**: 归档端点对已归档会话返回 409，这是预期行为，调用者不应将其视为错误。

5. **标题更新竞争**: 如果在用户重命名会话的同时服务器端也更新标题，可能存在竞争条件，最后一次更新胜出。

6. **source 字段**: 创建会话时 `source: 'remote-control'` 是必需的，用于区分不同入口点的会话。

7. **ValidateStatus**: 使用 `validateStatus: s => s < 500` 区分客户端错误（4xx）和服务器错误（5xx）。

8. **超时配置**: `archiveBridgeSession` 支持自定义超时（默认 10s），适应不同的网络环境。
