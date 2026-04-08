# teammateLayoutManager.ts — 队友布局管理

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/teammateLayoutManager.ts`  
**关联模块**: 窗格管理、后端检测  
**主要依赖**: `src/tools/AgentTool/agentColorManager.js`

## 功能概述

本文件管理 teammates 的颜色分配和窗格布局操作。提供 teammate 颜色分配和创建 teammate 窗格的统一接口。

## 核心内容详解

### 颜色管理

#### `assignTeammateColor(teammateId): AgentColorName`
为 teammate 分配唯一颜色：
- 从可用调色板中轮询分配
- 如果已分配，返回现有颜色
- 使用 `AGENT_COLORS` 数组循环分配

#### `getTeammateColor(teammateId): AgentColorName | undefined`
获取已分配的颜色（如果有）。

#### `clearTeammateColors()`
清除所有 teammate 颜色分配。
- 在团队清理时调用，为潜在新团队重置状态

### 窗格操作

#### `isInsideTmux(): Promise<boolean>`
检查当前是否在 tmux 会话中运行。

#### `createTeammatePaneInSwarmView(name, color): Promise<{ paneId: string; isFirstTeammate: boolean }>`
在 swarm 视图中创建新的 teammate 窗格：
- 自动检测适当的后端（tmux 或 iTerm2）
- 根据环境选择布局策略

**tmux 内部运行**:
- 使用 TmuxBackend 分割当前窗口
- 领导在左侧（30%），teammates 在右侧（70%）

**iTerm2 运行**（不在 tmux 中）:
- 使用 ITermBackend 原生 iTerm2 分割窗格

**tmux/iTerm2 外部运行**:
- 回退到 TmuxBackend 使用外部 claude-swarm 会话

#### `enablePaneBorderStatus(windowTarget?, useSwarmSocket?): Promise<void>`
为窗口启用窗格边框状态显示（显示窗格标题）。

#### `sendCommandToPane(paneId, command, useSwarmSocket?): Promise<void>`
向特定窗格发送命令执行。

## 设计要点

1. **颜色轮询**: 简单的轮询分配确保 teammates 有不同颜色
2. **后端自动检测**: `detectAndGetBackend()` 自动选择适当的后端
3. **布局策略**: 不同环境使用不同布局（有领导 vs 无领导）
4. **后端缓存**: `detectAndGetBackend()` 内部缓存，无需额外缓存

## 与其他文件的关系

- **backends/registry.ts**: 使用 `detectAndGetBackend`
- **backends/types.ts**: 使用 `PaneBackend` 类型
- **agentColorManager.ts**: 使用 `AGENT_COLORS` 和 `AgentColorName`
- **backends/detection.ts**: 使用 `isInsideTmux`

## 注意事项

- 颜色分配在会话期间持久化，不跨会话
- `isFirstTeammate` 影响布局策略（第一个 teammate 使用垂直分割）
- 窗格创建使用锁机制防止并行创建时的竞态条件
