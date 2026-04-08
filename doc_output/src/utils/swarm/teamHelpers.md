# teamHelpers.ts — 团队管理助手

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/teamHelpers.ts`  
**关联模块**: 团队管理、文件系统  
**主要依赖**: `zod`, `src/utils/envUtils.js`, `src/utils/git.js`

## 功能概述

本文件提供团队管理的核心辅助功能：
- 团队文件的读写操作
- 团队成员管理（添加、移除、状态更新）
- 团队目录和任务目录的清理
- Git worktree 的管理

## 核心内容详解

### 类型定义

```typescript
type TeamFile = {
  name: string
  description?: string
  createdAt: number
  leadAgentId: string
  leadSessionId?: string
  hiddenPaneIds?: string[]           // 当前隐藏的窗格 ID
  teamAllowedPaths?: TeamAllowedPath[]
  members: Array<{
    agentId: string
    name: string
    agentType?: string
    model?: string
    prompt?: string
    color?: string
    planModeRequired?: boolean
    joinedAt: number
    tmuxPaneId: string
    cwd: string
    worktreePath?: string
    sessionId?: string
    subscriptions: string[]
    backendType?: BackendType
    isActive?: boolean               // false 表示空闲
    mode?: PermissionMode
  }>
}

type TeamAllowedPath = {
  path: string
  toolName: string
  addedBy: string
  addedAt: number
}
```

### 核心函数

#### `sanitizeName(name): string`
清理名称用于 tmux 窗口名、worktree 路径和文件路径：
- 将所有非字母数字字符替换为连字符
- 转换为小写

#### `readTeamFile(teamName)` / `readTeamFileAsync(teamName)`
读取团队配置文件（同步/异步）：
- 路径: `~/.claude/teams/{teamName}/config.json`
- 返回 `TeamFile` 或 `null`（不存在时）

#### `removeTeammateFromTeamFile(teamName, identifier)`
从团队文件中移除 teammate：
- 支持通过 `agentId` 或 `name` 标识
- 返回是否成功移除

#### `setMemberMode(teamName, memberName, mode)`
设置团队成员的权限模式：
- 更新团队成员的 `mode` 字段
- 仅在实际值改变时才写入

#### `setMemberActive(teamName, memberName, isActive)`
设置团队成员的活跃状态：
- 用于标记 teammate 是否空闲
- 异步写入

#### `cleanupTeamDirectories(teamName)`
清理团队的目录和文件：
- 销毁 git worktree
- 删除团队目录（`~/.claude/teams/{teamName}/`）
- 删除任务目录（`~/.claude/tasks/{sanitizedName}/`）

#### `registerTeamForSessionCleanup(teamName)`
标记团队为会话创建，用于退出时清理：
- 存储在 `getSessionCreatedTeams()` Set 中
- `resetStateForTests()` 会清除，避免测试间泄漏

## 设计要点

1. **名称清理**: 统一的 `sanitizeName` 确保文件系统安全
2. **会话清理**: 使用 Set 跟踪会话创建的团队，优雅退出时自动清理
3. **Worktree 销毁**: 先尝试 `git worktree remove`，失败则回退到 `rm -rf`
4. **状态同步**: `syncTeammateMode` 将当前 teammate 的模式同步到配置文件

## 与其他文件的关系

- **spawnInProcess.ts**: 调用 `removeMemberByAgentId` 清理
- **teammateInit.ts**: 应用团队级允许路径
- **bootstrap/state.ts**: `getSessionCreatedTeams` 跟踪会话团队

## 注意事项

- `projectSettings` 被排除在危险模式权限提示之外（防止恶意项目 RCE）
- Worktree 清理可能失败，日志记录但继续执行
- `cleanupSessionTeams` 在 SIGINT/SIGTERM 时杀死孤立 teammate 窗格
