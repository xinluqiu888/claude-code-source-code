# PaneBackendExecutor.ts — 窗格后端执行器

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/backends/PaneBackendExecutor.ts`  
**关联模块**: 窗格后端适配、队友生命周期  
**主要依赖**: `src/utils/swarm/spawnUtils.js`, `src/utils/agentId.js`

## 功能概述

本文件将 `PaneBackend` 适配到 `TeammateExecutor` 接口，使基于窗格的后端（tmux、iTerm2）可以通过与进程内后端相同的抽象进行使用。

`PaneBackend` 处理低级窗格操作；`PaneBackendExecutor` 处理跨所有后端工作的高级队友生命周期操作。

## 核心内容详解

### 类定义

```typescript
class PaneBackendExecutor implements TeammateExecutor {
  readonly type: BackendType
  private backend: PaneBackend
  private context: ToolUseContext | null = null
  private spawnedTeammates: Map<string, { paneId: string; insideTmux: boolean }>
}
```

### 核心方法

#### `setContext(context)`
设置 `ToolUseContext` 用于 AppState 访问。必须在 `spawn()` 前调用。

#### `spawn(config)`
在窗格中生成 teammate：

1. **分配颜色**：如果未提供，分配 teammate 颜色
2. **创建窗格**：调用 `backend.createTeammatePaneInSwarmView()`
3. **启用边框状态**：如果是第一个 teammate 且在 tmux 中
4. **构建 CLI 命令**：
   - 获取二进制路径 (`getTeammateCommand()`)
   - 构建身份参数 (`--agent-id`, `--agent-name`, `--team-name` 等)
   - 构建继承标志 (`buildInheritedCliFlags()`)
   - 构建环境变量 (`buildInheritedEnvVars()`)
   - 组合为完整命令
5. **发送命令**：调用 `backend.sendCommandToPane()`
6. **跟踪 teammate**：存储在 `spawnedTeammates` Map 中
7. **注册清理**：在领导者退出时杀死所有窗格
8. **发送初始消息**：通过邮箱发送提示

#### `sendMessage(agentId, message)`
向窗格-based teammate 发送消息：
- 解析 `agentId` 获取 `agentName` 和 `teamName`
- 写入文件邮箱
- 所有 teammates 使用相同的邮箱机制

#### `terminate(agentId, reason)`
优雅地终止窗格-based teammate：
- 解析 `agentId`
- 创建关闭请求消息
- 通过邮箱发送给 teammate
- 不直接杀死窗格

#### `kill(agentId)`
强制关闭窗格：
- 在 `spawnedTeammates` 中查找窗格信息
- 调用 `backend.killPane()`
- 从 Map 中删除

#### `isActive(agentId)`
检查窗格-based teammate 是否仍然活跃：
- 在 `spawnedTeammates` 中查找
- 如果有记录则假设活跃
- 注意：窗格可能存在但内部进程可能已退出

## 设计要点

1. **适配器模式**: 包装 `PaneBackend` 以提供统一的 `TeammateExecutor` 接口
2. **命令构建**: 组合身份参数、继承标志和环境变量
3. **窗格跟踪**: 维护 `paneId` 和 `insideTmux` 用于后续操作
4. **清理注册**: 领导者退出时自动杀死窗格

## 与其他文件的关系

- **types.ts**: 实现 `TeammateExecutor` 接口
- **spawnUtils.ts**: 使用 `getTeammateCommand`, `buildInheritedCliFlags`, `buildInheritedEnvVars`
- **teammateLayoutManager.ts**: 使用 `assignTeammateColor`
- **teammateMailbox.ts**: 使用 `writeToMailbox`
- **cleanupRegistry.ts**: 注册退出时清理

## 注意事项

- 必须在调用 `spawn()` 前调用 `setContext()`
- `insideTmux` 影响使用哪个 socket（用户会话 vs 外部 swarm 会话）
- `terminate()` 是优雅的，通过邮箱发送请求
- `kill()` 是强制的，直接关闭窗格
- 窗格可能存在但进程已退出，`isActive` 可能返回误报
