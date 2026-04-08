# types.ts — 后端类型定义

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/backends/types.ts`  
**关联模块**: Swarm 后端系统  
**主要依赖**: `src/tools/AgentTool/agentColorManager.js`

## 功能概述

本文件定义 Swarm 后端系统的核心类型，包括后端类型、窗格后端接口、队友执行器接口等。为 pane-based（tmux/iTerm2）和 in-process 两种执行模式提供统一的抽象。

## 核心内容详解

### 后端类型

```typescript
type BackendType = 'tmux' | 'iterm2' | 'in-process'
type PaneBackendType = 'tmux' | 'iterm2'
type PaneId = string  // tmux: "%1", iTerm2: session UUID
```

### 窗格后端接口 (PaneBackend)

```typescript
interface PaneBackend {
  readonly type: BackendType
  readonly displayName: string
  readonly supportsHideShow: boolean
  
  isAvailable(): Promise<boolean>
  isRunningInside(): Promise<boolean>
  createTeammatePaneInSwarmView(name: string, color: AgentColorName): Promise<CreatePaneResult>
  sendCommandToPane(paneId: PaneId, command: string, useExternalSession?: boolean): Promise<void>
  setPaneBorderColor(paneId: PaneId, color: AgentColorName, useExternalSession?: boolean): Promise<void>
  setPaneTitle(paneId: PaneId, name: string, color: AgentColorName, useExternalSession?: boolean): Promise<void>
  enablePaneBorderStatus(windowTarget?: string, useExternalSession?: boolean): Promise<void>
  rebalancePanes(windowTarget: string, hasLeader: boolean): Promise<void>
  killPane(paneId: PaneId, useExternalSession?: boolean): Promise<boolean>
  hidePane(paneId: PaneId, useExternalSession?: boolean): Promise<boolean>
  showPane(paneId: PaneId, targetWindowOrPane: string, useExternalSession?: boolean): Promise<boolean>
}
```

### 队友执行器接口 (TeammateExecutor)

```typescript
interface TeammateExecutor {
  readonly type: BackendType
  isAvailable(): Promise<boolean>
  spawn(config: TeammateSpawnConfig): Promise<TeammateSpawnResult>
  sendMessage(agentId: string, message: TeammateMessage): Promise<void>
  terminate(agentId: string, reason?: string): Promise<boolean>
  kill(agentId: string): Promise<boolean>
  isActive(agentId: string): Promise<boolean>
}
```

### 配置和结果类型

```typescript
type TeammateSpawnConfig = {
  name: string
  teamName: string
  prompt: string
  color?: string
  planModeRequired?: boolean
  model?: string
  systemPrompt?: string
  systemPromptMode?: 'default' | 'replace' | 'append'
  worktreePath?: string
  parentSessionId: string
  permissions?: string[]
  allowPermissionPrompts?: boolean
}

type TeammateSpawnResult = {
  success: boolean
  agentId: string  // 格式: agentName@teamName
  error?: string
  abortController?: AbortController  // in-process 专用
  taskId?: string  // in-process 专用
  paneId?: PaneId  // pane-based 专用
}

type TeammateMessage = {
  text: string
  from: string
  color?: string
  timestamp?: string
  summary?: string  // 5-10 字预览
}

type CreatePaneResult = {
  paneId: PaneId
  isFirstTeammate: boolean
}

type BackendDetectionResult = {
  backend: PaneBackend
  isNative: boolean
  needsIt2Setup?: boolean
}
```

### 类型守卫

```typescript
function isPaneBackend(type: BackendType): type is 'tmux' | 'iterm2'
```

## 设计要点

1. **后端抽象**: `PaneBackend` 处理低级窗格操作，`TeammateExecutor` 处理高级队友生命周期
2. **统一消息**: 所有 teammates（pane 和 in-process）使用相同的邮箱机制
3. **灵活配置**: `TeammateSpawnConfig` 支持系统提示、模型覆盖、权限列表等
4. **结果区分**: `TeammateSpawnResult` 包含后端特定的字段（`taskId` vs `paneId`）

## 与其他文件的关系

- **agentColorManager.ts**: 使用 `AgentColorName`
- **TmuxBackend.ts**: 实现 `PaneBackend`
- **ITermBackend.ts**: 实现 `PaneBackend`
- **InProcessBackend.ts**: 实现 `TeammateExecutor`
- **PaneBackendExecutor.ts**: 将 `PaneBackend` 适配到 `TeammateExecutor`

## 注意事项

- `paneId` 格式因后端而异：tmux 使用 `%1` 格式，iTerm2 使用 UUID
- `isNative` 表示 teammates 是否显示在原生窗格中
- `needsIt2Setup` 用于 iTerm2 但未安装 it2 CLI 的情况
