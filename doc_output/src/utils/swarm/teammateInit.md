# teammateInit.ts — 队友初始化

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/teammateInit.ts`  
**关联模块**: 队友生命周期、权限系统  
**主要依赖**: `src/hooks/sessionHooks.js`, `src/utils/teammateMailbox.js`

## 功能概述

本文件处理作为 swarm 中 teammates 运行的 Claude Code 实例的初始化。注册 Stop hook 以在 teammate 变为空闲时通知团队领导。

## 核心内容详解

### 核心函数

#### `initializeTeammateHooks(setAppState, sessionId, teamInfo)`
初始化 swarm 中运行的 teammate 的 hooks：
- 在会话启动早期、AppState 可用后调用
- 注册 Stop hook，在会话停止时向团队领导发送空闲通知

参数：
```typescript
{
  setAppState: (updater: (prev: AppState) => AppState) => void
  sessionId: string
  teamInfo: { teamName: string; agentId: string; agentName: string }
}
```

### 初始化流程

1. **读取团队文件**: 获取领导 ID 和团队范围允许路径
2. **应用团队允许路径**: 如果存在，应用到工具权限上下文
   - 绝对路径（以 `/` 开头）: 添加 `//path/**` 模式
   - 相对路径: 添加 `path/**` 模式
3. **查找领导名称**: 从团队成员数组中获取
4. **跳过领导**: 如果当前 agent 是领导，跳过 hook 注册
5. **注册 Stop hook**: 在会话停止时发送空闲通知

### Stop Hook 行为

当会话停止时：
1. 在团队配置中将此 teammate 标记为空闲（fire-and-forget）
2. 向领导发送空闲通知：
   ```typescript
   createIdleNotification(agentName, {
     idleReason: 'available',
     summary: getLastPeerDmSummary(messages),
   })
   ```
3. 使用 agent 名称（非 UUID）发送到领导的邮箱
4. 10 秒超时确保在进程关闭前完成写入

## 设计要点

1. **早期初始化**: 在 AppState 可用后尽早调用，确保心跳和其他功能正常工作
2. **权限继承**: 自动应用团队范围的允许路径规则
3. **领导跳过**: 领导不需要向自己发送空闲通知
4. **超时保护**: 10 秒超时防止进程关闭前未完成写入

## 与其他文件的关系

- **teamHelpers.ts**: 使用 `readTeamFile` 和 `setMemberActive`
- **teammateMailbox.ts**: 使用 `writeToMailbox` 和 `createIdleNotification`
- **sessionHooks.ts**: 使用 `addFunctionHook` 注册 Stop hook
- **permissions/permissionSetup.ts**: 使用 `applyPermissionUpdate`

## 注意事项

- 团队成员可能已从团队文件中移除（如被删除）
- 团队允许路径在 teammate 启动时应用，后续变更需要重启
- 空闲通知使用 `void` 调用，不等待完成
