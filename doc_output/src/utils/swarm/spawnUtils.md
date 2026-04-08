# spawnUtils.ts — 队友进程启动工具

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/spawnUtils.ts`  
**关联模块**: 队友管理、权限系统、CLI 参数  
**主要依赖**: `src/bootstrap/state.js`, `src/utils/permissions/PermissionMode.js`

## 功能概述

本文件提供跨不同后端（tmux/iTerm2）启动 teammates 的共享工具：
- 获取 teammate 启动命令
- 构建要传播的 CLI 标志
- 构建要继承的环境变量

## 核心内容详解

### 核心函数

#### `getTeammateCommand(): string`
获取用于启动 teammate 进程的命令：
- 优先使用 `TEAMMATE_COMMAND_ENV_VAR` 环境变量
- 默认使用当前进程可执行路径
- 捆绑模式使用 `process.execPath`，否则使用 `process.argv[1]`

#### `buildInheritedCliFlags(options?): string`
构建要传播给 teammate 的 CLI 标志：

| 传播的标志 | 条件 |
|-----------|------|
| `--dangerously-skip-permissions` | 权限模式为 `bypassPermissions` 且非计划模式 |
| `--permission-mode acceptEdits` | 权限模式为 `acceptEdits` |
| `--model <model>` | 通过 CLI 显式设置模型 |
| `--settings <path>` | 通过 CLI 设置 settings 路径 |
| `--plugin-dir <dir>` | 每个内联插件目录 |
| `--teammate-mode <mode>` | 传播 teammate 模式 |
| `--chrome` / `--no-chrome` | Chrome 标志覆盖 |

#### `buildInheritedEnvVars(): string`
构建要转发给 teammate 的环境变量：

**必需变量**:
- `CLAUDECODE=1`
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

**API 提供商选择**:
- `CLAUDE_CODE_USE_BEDROCK`
- `CLAUDE_CODE_USE_VERTEX`
- `CLAUDE_CODE_USE_FOUNDRY`
- `ANTHROPIC_BASE_URL`

**配置覆盖**:
- `CLAUDE_CONFIG_DIR`
- `CLAUDE_CODE_REMOTE`
- `CLAUDE_CODE_REMOTE_MEMORY_DIR`

**代理设置**:
- `HTTPS_PROXY`, `https_proxy`
- `HTTP_PROXY`, `http_proxy`
- `NO_PROXY`, `no_proxy`
- `SSL_CERT_FILE`, `NODE_EXTRA_CA_CERTS`
- `REQUESTS_CA_BUNDLE`, `CURL_CA_BUNDLE`

## 设计要点

1. **权限安全**: 计划模式优先于绕过权限，确保队友也需遵守计划审批流程
2. **环境隔离**: tmux 可能启动新的登录 shell 不继承父进程环境，显式转发关键变量
3. **Cowork 兼容**: 转发 `CLAUDE_CODE_REMOTE_MEMORY_DIR` 避免内存功能在 CCR 中意外开启

## 与其他文件的关系

- **constants.ts**: 导入环境变量名称常量
- **PaneBackendExecutor.ts**: 调用 `buildInheritedCliFlags` 和 `buildInheritedEnvVars`
- **teammateModeSnapshot.ts**: 获取 teammate 模式

## 注意事项

- `TEAMMATE_ENV_VARS` 列表需要与 API 配置同步维护
- 代理变量转发确保 teammates 通过父进程的 MITM 中继路由
- 从家目录运行安装命令避免读取项目级 pip.conf/uv.toml（安全风险）
