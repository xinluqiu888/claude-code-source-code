# constants.ts — Swarm 常量定义

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/constants.ts`  
**关联模块**: Swarm 团队管理、tmux 集成  
**主要依赖**: 无

## 功能概述

本文件定义了 Agent Swarm（智能体集群）功能的核心常量，包括：
- 团队领导和会话名称常量
- tmux 相关常量
- 环境变量名称
- 用于隔离 Swarm 操作的 socket 名称生成

## 核心内容详解

### 常量定义

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `TEAM_LEAD_NAME` | `'team-lead'` | 团队领导（Leader）的默认名称 |
| `SWARM_SESSION_NAME` | `'claude-swarm'` | 外部 Swarm 会话名称 |
| `SWARM_VIEW_WINDOW_NAME` | `'swarm-view'` | Swarm 视图窗口名称 |
| `TMUX_COMMAND` | `'tmux'` | tmux 命令名 |
| `HIDDEN_SESSION_NAME` | `'claude-hidden'` | 隐藏会话名称（用于隐藏窗格） |

### 环境变量

| 变量名 | 常量 | 说明 |
|--------|------|------|
| `CLAUDE_CODE_TEAMMATE_COMMAND` | `TEAMMATE_COMMAND_ENV_VAR` | 覆盖队友进程启动命令 |
| `CLAUDE_CODE_AGENT_COLOR` | `TEAMMATE_COLOR_ENV_VAR` | 设置队友颜色标识 |
| `CLAUDE_CODE_PLAN_MODE_REQUIRED` | `PLAN_MODE_REQUIRED_ENV_VAR` | 要求队友进入计划模式 |

### 函数

#### `getSwarmSocketName(): string`
获取外部 Swarm 会话的 socket 名称：
- 使用独立的 socket 隔离 Swarm 操作与用户 tmux 会话
- 包含进程 PID 确保多个 Claude 实例不冲突
- 格式: `claude-swarm-${process.pid}`

## 设计要点

1. **Socket 隔离**: 当用户不在 tmux 中时，使用独立 socket 避免干扰用户的 tmux 会话
2. **PID 命名**: 包含进程 ID 确保同一系统上多个 Claude Code 实例不冲突
3. **环境变量配置**: 通过环境变量支持灵活的部署和测试场景

## 与其他文件的关系

- **TmuxBackend.ts**: 使用 `getSwarmSocketName()` 创建外部会话
- **spawnUtils.ts**: 使用 `TEAMMATE_COMMAND_ENV_VAR` 等环境变量
- **teamHelpers.ts**: 使用 `TEAM_LEAD_NAME`

## 注意事项

- 修改常量可能影响 tmux 会话的识别和管理
- `HIDDEN_SESSION_NAME` 用于实现窗格隐藏功能
- 环境变量在子进程中继承， teammates 可读取这些变量
