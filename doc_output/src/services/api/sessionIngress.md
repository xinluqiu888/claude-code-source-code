# sessionIngress.ts — 会话日志持久化服务

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/api/sessionIngress.ts`
- **所属模块**: API Service
- **功能类型**: 会话日志存储和检索

## 功能概述

该模块管理会话日志的持久化存储，支持通过 JWT 令牌将日志条目追加到远程存储，以及从远程存储获取会话日志用于恢复（Teleport）。

## 核心内容详解

### 常量配置

```typescript
MAX_RETRIES = 10                    // 最大重试次数
BASE_DELAY_MS = 500                 // 基础延迟
```

### 核心状态

#### `lastUuidMap: Map<string, UUID>`
每个会话的最后 UUID 映射，用于乐观并发控制。

#### `sequentialAppendBySession: Map<string, Function>`
每会话的顺序包装器映射，确保日志追加的顺序执行。

### 主要函数

#### `appendSessionLog(sessionId, entry, url): Promise<boolean>`
追加日志条目到会话存储。

**认证：**
- 使用 `getSessionIngressAuthToken()` 获取 JWT 令牌
- 请求头：`Authorization: Bearer {token}`

**顺序控制：**
- 每会话使用 `sequential()` 包装器
- 防止并发写入导致的状态冲突

**内部实现：** `appendSessionLogImpl`

**重试逻辑：**
- 指数退避（最大 10 次）
- 409 冲突处理（详见下文）
- 401 立即失败（非重试）

#### `getSessionLogs(sessionId, url): Promise<Entry[] | null>`
获取会话日志（JWT 认证）。

**返回：**
- 成功 — 日志条目数组
- 404 — 空数组（无现有日志）
- 401 — 抛出错误（令牌过期）
- 其他 — null

**副作用：**
- 更新 `lastUuidMap` 为最后条目的 UUID

#### `getSessionLogsViaOAuth(sessionId, accessToken, orgUUID): Promise<Entry[] | null>`
通过 OAuth 获取会话日志（用于 Teleport）。

**端点：**
```
GET /v1/session_ingress/session/{sessionId}
```

#### `getTeleportEvents(sessionId, accessToken, orgUUID): Promise<Entry[] | null>`
通过 CCR v2 Sessions API 获取 Teleport 事件。

**端点：**
```
GET /v1/code/sessions/{sessionId}/teleport-events
```

**分页：**
- 服务器默认 500/页，最大 1000
- 使用 `next_cursor` 游标
- 最大 100 页保护（10 万事件）

**事件结构：**
```typescript
{
  event_id: string
  event_type: string
  is_compaction: boolean
  payload: Entry | null
  created_at: string
}
```

### 409 冲突处理

当收到 409 状态码时的处理逻辑：

1. **检查条目是否已存在**
   - 如果 `x-last-uuid` 头等于当前条目 UUID → 已存储，恢复状态

2. **采用服务器的 last UUID**
   - 从 `x-last-uuid` 头获取
   - 更新 `lastUuidMap`
   - 重试当前条目

3. **重新获取会话**
   - 如果服务器未返回头 → 调用 `fetchSessionLogsFromUrl`
   - 查找最后 UUID
   - 如果找到 → 采用并重试
   - 未找到 → 放弃（并发修改无法解决）

### 辅助函数

#### `getOrCreateSequentialAppend(sessionId): Function`
获取或创建会话的顺序包装器。

#### `fetchSessionLogsFromUrl(sessionId, url, headers): Promise<Entry[] | null>`
从 URL 获取会话日志的共享实现。

**查询参数：**
- `after_last_compact=true`（如果 `CLAUDE_AFTER_LAST_COMPACT` 设置）

#### `findLastUuid(logs): UUID | undefined`
反向查找最后条目的 UUID（跳过无 UUID 的条目类型）。

### 状态清理

#### `clearSession(sessionId): void`
清除特定会话的缓存状态。

#### `clearAllSessions(): void`
清除所有会话的缓存状态。

**使用场景：**
- `/clear` 命令后释放子 Agent 会话条目

## 设计要点

1. **乐观并发控制** — 使用 Last-Uuid 头实现线性化追加
2. **顺序保证** — 每会话串行处理，防止竞态条件
3. **冲突恢复** — 智能 409 处理，支持服务器状态采用
4. **分页支持** — Teleport 事件支持大会话的分页获取
5. **多种认证** — 支持 JWT 和 OAuth 两种认证方式

## 与其他文件的关系

- **被调用**: 会话管理、Teleport 功能
- **依赖**: `../../utils/sessionIngressAuth.ts` 获取 JWT
- **关联**: `../../utils/sequential.ts` 顺序执行工具

## 注意事项

- 409 冲突通常由进程被杀但请求仍在飞行中导致
- 分页有 100 页保护，防止无限循环
- `payload` 可能为 null（threadstore 非通用事件或加密失败）
- 401 错误会抛出用户友好消息（"请运行 /login 重新登录"）
