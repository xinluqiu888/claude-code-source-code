# permissionSync.ts — Swarm 权限同步

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/permissionSync.ts`  
**关联模块**: 权限系统、队友邮箱  
**主要依赖**: `zod`, `src/utils/lockfile.js`, `src/utils/teammateMailbox.js`

## 功能概述

本文件提供 Agent Swarm 中跨多个 Agent 协调权限提示的基础设施：
- Worker agent 将权限请求转发给团队领导
- 领导通过 UI 批准或拒绝
- 使用基于文件的权限系统和邮箱消息传递

## 核心内容详解

### 类型定义

```typescript
type SwarmPermissionRequest = {
  id: string                     // 唯一请求标识
  workerId: string               // Worker 的 CLAUDE_CODE_AGENT_ID
  workerName: string             // Worker 的 CLAUDE_CODE_AGENT_NAME
  workerColor?: string           // Worker 的颜色
  teamName: string               // 团队名称
  toolName: string               // 需要权限的工具名
  toolUseId: string              // 原始 toolUseID
  description: string            // 工具使用描述
  input: Record<string, unknown> // 序列化工具输入
  permissionSuggestions: unknown[]
  status: 'pending' | 'approved' | 'rejected'
  resolvedBy?: 'worker' | 'leader'
  resolvedAt?: number
  feedback?: string              // 拒绝反馈
  updatedInput?: Record<string, unknown>
  permissionUpdates?: unknown[]
  createdAt: number
}

type PermissionResolution = {
  decision: 'approved' | 'rejected'
  resolvedBy: 'worker' | 'leader'
  feedback?: string
  updatedInput?: Record<string, unknown>
  permissionUpdates?: PermissionUpdate[]
}
```

### 核心函数

#### `writePermissionRequest(request)`
将权限请求写入待处理目录：
- 使用文件锁确保原子写入
- 路径: `~/.claude/teams/{teamName}/permissions/pending/{id}.json`

#### `readPendingPermissions(teamName)`
读取团队的所有待处理权限请求：
- 按创建时间排序（最旧优先）
- 解析并验证每个请求文件

#### `readResolvedPermission(requestId, teamName)`
读取已解决的权限请求：
- 路径: `~/.claude/teams/{teamName}/permissions/resolved/{id}.json`
- 返回 `null` 表示尚未解决

#### `resolvePermission(requestId, resolution, teamName)`
解决权限请求：
- 更新请求状态、解决者、时间戳
- 写入已解决目录
- 从待处理目录删除
- 使用文件锁保护

#### `sendPermissionRequestViaMailbox(request)`
通过邮箱发送权限请求（新邮箱方式）：
- 查找领导名称
- 创建权限请求消息
- 写入领导的邮箱

#### `sendPermissionResponseViaMailbox(workerName, resolution, requestId, teamName)`
通过邮箱发送权限响应：
- 创建权限响应消息
- 写入 worker 的邮箱

### 沙箱权限邮箱系统

专门的沙箱网络访问权限请求：
- `sendSandboxPermissionRequestViaMailbox(host, requestId, teamName)`
- `sendSandboxPermissionResponseViaMailbox(workerName, requestId, host, allow, teamName)`

## 设计要点

1. **文件锁机制**: 使用 `lockfile.lock` 确保并发安全
2. **目录结构**:
   ```
   ~/.claude/teams/{teamName}/permissions/
   ├── pending/      # 待处理请求
   └── resolved/     # 已解决请求
   ```
3. **邮箱集成**: 新系统使用邮箱消息替代直接文件操作
4. **垃圾回收**: `cleanupOldResolutions` 定期清理旧的已解决请求

## 与其他文件的关系

- **teammateMailbox.ts**: 使用邮箱消息传递
- **lockfile.ts**: 文件锁定
- **inProcessRunner.ts**: 处理权限响应

## 注意事项

- 权限请求文件使用 JSON 格式，便于调试
- `isTeamLeader` 检查 agentId 是否存在或是否为 'team-lead'
- 最大清理年龄默认为 1 小时
