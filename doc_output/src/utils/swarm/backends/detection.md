# detection.ts — 后端检测

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/backends/detection.ts`  
**关联模块**: 后端检测、环境识别  
**主要依赖**: `src/utils/env.js`, `src/utils/execFileNoThrow.js`

## 功能概述

本文件提供 tmux 和 iTerm2 环境的检测功能：
- 检测是否在 tmux 会话中运行
- 检测是否在 iTerm2 中运行
- 检测 tmux 和 it2 CLI 是否可用
- 捕获原始环境变量（避免后续覆盖）

## 核心内容详解

### 环境变量捕获

在模块加载时捕获原始环境变量（避免 Shell.ts 后续覆盖）：
- `ORIGINAL_USER_TMUX`: 用户启动 Claude 时的 `TMUX` 值
- `ORIGINAL_TMUX_PANE`: 领导者的 tmux 窗格 ID（`TMUX_PANE`）

### 核心函数

#### `isInsideTmuxSync(): boolean`
同步检查是否在 tmux 会话中：
- 使用模块加载时捕获的 `ORIGINAL_USER_TMUX`
- 比异步版本更快，用于同步上下文

#### `isInsideTmux(): Promise<boolean>`
异步检查是否在 tmux 会话中：
- 缓存结果（进程生命周期内不变）
- 仅检查 `TMUX` 环境变量
- **不**运行 `tmux display-message`（因为这会在任何 tmux 服务器运行时成功）

#### `getLeaderPaneId(): string | null`
获取模块加载时捕获的领导者 tmux 窗格 ID。

#### `isTmuxAvailable(): Promise<boolean>`
检查系统是否安装了 tmux：
- 运行 `tmux -V` 测试
- 返回退出码是否为 0

#### `isInITerm2(): boolean`
检查是否在 iTerm2 中运行：
- 检查 `TERM_PROGRAM === 'iTerm.app'`
- 检查 `ITERM_SESSION_ID` 是否存在
- 检查 `env.terminal === 'iTerm.app'`
- 缓存结果

#### `isIt2CliAvailable(): Promise<boolean>`
检查 it2 CLI 工具是否可用且能连接到 iTerm2 Python API：
- 运行 `it2 session list`（非 `--version`，因为后者即使 Python API 禁用时也成功）
- 返回退出码是否为 0

#### `resetDetectionCache(): void`
重置所有缓存的检测结果（用于测试）。

## 设计要点

1. **原始值捕获**: 在模块加载时捕获 `TMUX` 和 `TMUX_PANE`，避免后续覆盖
2. **不依赖 tmux 命令**: `isInsideTmux` 仅检查环境变量，避免误判
3. **iTerm2 多指标**: 使用多种方法检测 iTerm2，提高可靠性
4. **it2 功能测试**: 使用 `session list` 而非 `version`，确保 Python API 可用

## 与其他文件的关系

- **env.ts**: 使用 `env.terminal` 检测终端类型
- **registry.ts**: 使用检测函数选择后端
- **TmuxBackend.ts**: 使用 `getLeaderPaneId()` 获取领导者窗格

## 注意事项

- `isInsideTmux()` 仅在用户从 tmux 会话启动 Claude 时返回 true
- `isTmuxAvailable()` 检查 tmux 是否安装，不检查是否在 tmux 中
- `isInITerm2()` 可能返回 true 即使 it2 CLI 未安装
- 缓存机制确保多次调用高效
