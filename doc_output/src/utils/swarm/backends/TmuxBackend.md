# TmuxBackend.ts — Tmux 后端实现

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/backends/TmuxBackend.ts`  
**关联模块**: 窗格管理、tmux 集成  
**主要依赖**: `src/utils/execFileNoThrow.js`

## 功能概述

本文件实现使用 tmux 进行窗格管理的 `PaneBackend`：
- 在 tmux 内部运行时：分割当前窗口添加 teammates
- 在 tmux 外部运行时：创建独立的 `claude-swarm` 会话
- 领导者窗格保持在左侧（30%），teammates 在右侧（70%）

## 核心内容详解

### 类定义

```typescript
class TmuxBackend implements PaneBackend {
  readonly type = 'tmux'
  readonly displayName = 'tmux'
  readonly supportsHideShow = true
}
```

### 布局策略

**在 tmux 中运行**（有领导者）：
- 第一个 teammate：从领导者窗格水平分割（`-h`），占 70%
- 后续 teammates：从现有 teammate 分割，垂直/水平交替

**在 tmux 外运行**（无领导者）：
- 创建 `claude-swarm` 会话和 `swarm-view` 窗口
- 所有 teammates 均等分布（平铺布局）

### 核心方法

#### `createTeammatePaneInSwarmView(name, color)`
创建 teammate 窗格：
- 使用锁机制防止并行创建竞态条件
- 根据是否在 tmux 中调用不同的创建方法
- 设置窗格边框颜色和标题
- 等待 shell 初始化（200ms）

#### `sendCommandToPane(paneId, command, useExternalSession)`
向窗格发送命令执行：
- 使用 `send-keys` 和 `Enter`
- `useExternalSession` 时连接外部 socket

#### `setPaneBorderColor(paneId, color, useExternalSession)`
设置窗格边框颜色：
- 使用 `select-pane -P` 设置前景色
- 使用 `set-option -p` 设置窗格边框样式
- 使用 `set-option -p` 设置活动窗格边框样式

#### `setPaneTitle(paneId, name, color, useExternalSession)`
设置窗格标题：
- 使用 `select-pane -T` 设置标题
- 使用 `set-option -p` 设置边框格式（彩色标题）

#### `enablePaneBorderStatus(windowTarget?, useExternalSession)`
启用窗格边框状态（显示标题）：
- 使用 `set-option -w pane-border-status top`

#### `killPane(paneId, useExternalSession)`
关闭窗格：
- 使用 `kill-pane -t`

#### `hidePane(paneId, useExternalSession)`
隐藏窗格（移动到隐藏会话）：
- 创建 `claude-hidden` 会话（如果不存在）
- 使用 `break-pane -d` 移动到隐藏会话

#### `showPane(paneId, targetWindowOrPane, useExternalSession)`
显示隐藏的窗格：
- 使用 `join-pane -h` 移动回目标窗口
- 重新应用 `main-vertical` 布局

### 辅助方法

- `getCurrentPaneId()`: 获取领导者窗格 ID（使用 `TMUX_PANE`）
- `getCurrentWindowTarget()`: 获取当前窗口目标（session:window）
- `getCurrentWindowPaneCount()`: 获取窗口中的窗格数
- `rebalancePanesWithLeader()`: 重新平衡有领导者的窗格
- `rebalancePanesTiled()`: 重新平衡平铺布局的窗格

### 颜色映射

```typescript
const tmuxColors: Record<AgentColorName, string> = {
  red: 'red',
  blue: 'blue',
  green: 'green',
  yellow: 'yellow',
  purple: 'magenta',
  orange: 'colour208',
  pink: 'colour205',
  cyan: 'cyan',
}
```

## 设计要点

1. **Socket 切换**: `runTmuxInUserSession` vs `runTmuxInSwarm` 处理内外会话
2. **锁机制**: `acquirePaneCreationLock` 防止并行创建竞态条件
3. **领导者窗格缓存**: `cachedLeaderWindowTarget` 避免重复查询
4. **Shell 初始化延迟**: 200ms 延迟确保 shell 就绪

## 与其他文件的关系

- **types.ts**: 实现 `PaneBackend` 接口
- **registry.ts**: 注册后端类
- **constants.ts**: 使用 `getSwarmSocketName()`, `SWARM_SESSION_NAME`
- **detection.ts**: 使用 `isInsideTmux`, `getLeaderPaneId`

## 注意事项

- tmux 3.2+ 支持窗格特定选项（`-p`）
- 窗格标题在边框状态启用时显示
- 隐藏窗格功能需要 `claude-hidden` 会话
- 颜色使用 tmux 内置颜色名或 256 色（colourN）
