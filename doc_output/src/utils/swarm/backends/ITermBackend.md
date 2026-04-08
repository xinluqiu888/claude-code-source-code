# ITermBackend.ts — iTerm2 后端实现

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/backends/ITermBackend.ts`  
**关联模块**: 窗格管理、iTerm2 集成  
**主要依赖**: `src/utils/execFileNoThrow.js`

## 功能概述

本文件实现使用 iTerm2 原生分屏窗格的 `PaneBackend`：
- 通过 it2 CLI 工具控制 iTerm2
- 领导者在左侧（30%），teammates 在右侧垂直堆叠
- 不支持隐藏/显示窗格（it2 限制）

## 核心内容详解

### 类定义

```typescript
class ITermBackend implements PaneBackend {
  readonly type = 'iterm2'
  readonly displayName = 'iTerm2'
  readonly supportsHideShow = false
}
```

### 布局策略

- **第一个 teammate**：从领导者会话垂直分割（`-v`），占 70%
- **后续 teammates**：从最后一个 teammate 会话水平分割
- 使用 `-s` 标志显式指定要分割的会话，确保正确布局

### 核心方法

#### `createTeammatePaneInSwarmView(name, color)`
创建 teammate 窗格：
- 使用锁机制防止并行创建竞态条件
- 确定分割参数（垂直/水平、目标会话）
- 运行 `it2 session split` 创建窗格
- 解析输出获取新会话 ID
- 错误恢复：如果目标会话已死，修剪并重试

#### `sendCommandToPane(paneId, command, useExternalSession)`
向窗格发送命令：
- 使用 `it2 session run -s <paneId> <command>`
- 自动添加换行符执行

#### `setPaneBorderColor()`, `setPaneTitle()`, `enablePaneBorderStatus()`
这些操作在 iTerm2 中为 no-op：
- 每个 it2 调用都会生成 Python 进程，性能开销大
- iTerm2 在标签中自动显示标题
- 跳过这些操作以提高性能

#### `killPane(paneId, useExternalSession)`
关闭窗格：
- 使用 `it2 session close -f -s <paneId>`
- `-f` 标志强制关闭（绕过 "关闭前确认" 偏好设置）

#### `hidePane()`, `showPane()`
不支持：
- iTerm2 没有等同于 tmux 的 `break-pane`/`join-pane` 功能
- 记录调试日志并返回 `false`

### 辅助函数

#### `parseSplitOutput(output): string`
解析 `it2 session split` 输出：
- 格式：`Created new pane: <session-id>`
- 提取会话 ID

#### `getLeaderSessionId(): string | null`
从 `ITERM_SESSION_ID` 环境变量获取领导者会话 ID：
- 格式：`wXtYpZ:UUID`
- 提取冒号后的 UUID 部分

### 会话跟踪

```typescript
const teammateSessionIds: string[] = []  // 跟踪 teammate 会话 ID
let firstPaneUsed = false                // 是否已使用第一个窗格
let paneCreationLock: Promise<void>      // 创建锁
```

## 设计要点

1. **锁机制**: `acquirePaneCreationLock` 确保顺序创建
2. **死会话恢复**: 如果目标会话已关闭，修剪列表并重试
3. **显式目标**: 使用 `-s` 标志确保正确分割
4. **性能优化**: 跳过颜色和标题设置（每个 it2 调用开销大）

## 与其他文件的关系

- **types.ts**: 实现 `PaneBackend` 接口
- **registry.ts**: 注册后端类
- **detection.ts**: 使用 `isInITerm2`, `isIt2CliAvailable`

## 注意事项

- it2 CLI 需要安装 `pip install it2`
- Python API 必须在 iTerm2 设置中启用
- 不支持隐藏/显示窗格
- 会话 ID 仅在分割时使用 `-s` 标志从特定会话分割时有效
- 用户关闭窗格（Cmd+W/X）后，`teammateSessionIds` 中的条目变为无效，需要修剪
