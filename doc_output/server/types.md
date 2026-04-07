# types.ts — 服务器相关类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/server/types.ts`
- **类型**: TypeScript 模块
- **作用**: 定义服务器模块共享的类型和Schema

## 功能概述

本模块集中定义了与Claude Code服务器功能相关的所有类型，包括服务器配置、会话状态、会话索引等。这些类型用于直连服务器、会话管理和持久化。

## 核心内容详解

### 连接响应Schema

```typescript
export const connectResponseSchema = lazySchema(() =>
  z.object({
    session_id: z.string(),
    ws_url: z.string(),
    work_dir: z.string().optional(),
  }),
)
```

用于验证`createDirectConnectSession`的响应。

### 服务器配置

```typescript
export type ServerConfig = {
  port: number           // 服务器端口
  host: string           // 绑定主机
  authToken: string      // 认证令牌
  unix?: string          // Unix域套接字路径
  idleTimeoutMs?: number // 分离会话空闲超时（ms，0=永不过期）
  maxSessions?: number   // 最大并发会话数
  workspace?: string     // 未指定cwd时的默认工作区
}
```

### 会话状态

```typescript
export type SessionState =
  | 'starting'   // 正在启动
  | 'running'    // 运行中
  | 'detached'   // 已分离（后台运行）
  | 'stopping'   // 正在停止
  | 'stopped'    // 已停止
```

### 会话信息

```typescript
export type SessionInfo = {
  id: string              // 会话ID
  status: SessionState    // 当前状态
  createdAt: number       // 创建时间戳
  workDir: string         // 工作目录
  process: ChildProcess | null  // 子进程（可能为null）
  sessionKey?: string     // 可选会话密钥
}
```

### 会话索引

会话索引用于持久化会话元数据，支持跨服务器重启恢复会话。

```typescript
export type SessionIndexEntry = {
  sessionId: string           // 服务器分配的会话ID
  transcriptSessionId: string // Claude转录会话ID（用于--resume）
  cwd: string                 // 工作目录
  permissionMode?: string     // 权限模式
  createdAt: number           // 创建时间
  lastActiveAt: number        // 最后活动时间
}

export type SessionIndex = Record<string, SessionIndexEntry>
```

**持久化位置**: `~/.claude/server-sessions.json`

## 设计要点

1. **懒加载Schema**: 使用`lazySchema`延迟创建Zod schema，避免启动时加载
2. **可选字段**: 使用`?`标记可选字段，明确哪些可以缺失
3. **状态机**: SessionState形成完整的状态转换图
4. **双重ID**: sessionId和transcriptSessionId区分服务器会话和Claude会话

## 与其他文件的关系

- **被 createDirectConnectSession.ts 使用**: 导入connectResponseSchema
- **被 directConnectManager.ts 使用**: 共享DirectConnectConfig
- **被服务器实现使用**: 服务器端代码使用这些类型

## 注意事项

1. **SessionIndex持久化**: 会话索引保存到用户主目录，支持重启后恢复
2. **transcriptSessionId**: 与`--resume`使用的Claude会话ID相同
3. **permissionMode**: 可选的权限模式配置
4. **ChildProcess**: 服务器端使用，客户端代码中可能为null
