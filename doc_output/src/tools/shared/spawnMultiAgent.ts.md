# spawnMultiAgent.ts — 多智能体生成共享模块

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/shared/spawnMultiAgent.ts`
- **类型**: TypeScript 共享模块
- **功能**: 提供队友生成的共享逻辑，供 TeammateTool 和 AgentTool 使用

## 核心内容详解

### 导出函数

```typescript
// 模型解析
export function resolveTeammateModel(
  inputModel: string | undefined,
  leaderModel: string | null
): string

// 名称生成
export async function generateUniqueTeammateName(
  baseName: string,
  teamName: string | undefined
): Promise<string>

// 主要入口
export async function spawnTeammate(
  config: SpawnTeammateConfig,
  context: ToolUseContext
): Promise<{ data: SpawnOutput }>
```

### 类型定义

```typescript
export type SpawnOutput = {
  teammate_id: string
  agent_id: string
  agent_type?: string
  model?: string
  name: string
  color?: string
  tmux_session_name: string
  tmux_window_name: string
  tmux_pane_id: string
  team_name?: string
  is_splitpane?: boolean
  plan_mode_required?: boolean
}

export type SpawnTeammateConfig = {
  name: string
  prompt: string
  team_name?: string
  cwd?: string
  use_splitpane?: boolean
  plan_mode_required?: boolean
  model?: string
  agent_type?: string
  description?: string
  invokingRequestId?: string
}
```

### 模型解析

#### resolveTeammateModel
处理 `inherit` 别名和默认模型：

```typescript
// 'inherit' → 负责人模型
// undefined → 默认模型（Opus 或配置的值）
// 其他 → 直接使用
```

### 生成流程

#### handleSpawn
检测并选择生成模式：
1. **In-process 模式**: `isInProcessEnabled()` 时启用
2. **Pane 回退**: 无可用后端时回退到 in-process
3. **Split-pane 模式**: 默认模式（tmux/iTerm2）
4. **独立窗口模式**: `use_splitpane === false`

#### handleSpawnSplitPane
分屏视图生成：
- **tmux 内**: 在当前窗口分割（负责人左侧，队友右侧）
- **iTerm2**: 使用原生 iTerm2 分割
- **外部**: 创建 claude-swarm 会话，平铺布局

#### handleSpawnSeparateWindow
独立窗口生成（遗留模式）：
- 为每个队友创建独立的 tmux 窗口

#### handleSpawnInProcess
同进程内生成：
- 使用 AsyncLocalStorage
- 在同一 Node.js 进程中运行

### CLI 标志传播

#### buildInheritedCliFlags
传播给队友的 CLI 标志：

| 标志 | 条件 |
|------|------|
| `--dangerously-skip-permissions` | 绕过权限模式（非计划模式） |
| `--permission-mode acceptEdits` | 接受编辑模式 |
| `--permission-mode auto` | 自动模式 |
| `--model {model}` | 显式设置的模型 |
| `--settings {path}` | 设置文件路径 |
| `--plugin-dir {dir}` | 插件目录 |
| `--chrome` / `--no-chrome` | Chrome 标志 |

### 团队文件管理

#### 注册队友
1. 读取团队配置文件
2. 添加新成员信息
3. 写入更新后的配置

#### 发送初始指令
- tmux 队友：通过 mailbox 发送
- in-process 队友：直接传递

### 后台任务注册

#### registerOutOfProcessTeammateTask
为 tmux/iTerm2 队友注册后台任务：
- 使队友在任务丸/对话框中可见
- 设置中止监听器以终止 pane

## 设计要点

### 后端检测
- `detectAndGetBackend()` 自动检测可用后端
- 支持 tmux、iTerm2、in-process

### 名称唯一性
- 检查团队中现有名称
- 重复时添加数字后缀（如 `tester-2`）

### 错误处理
- iTerm2 未设置时显示设置提示
- 用户可以选择安装 it2、使用 tmux 或取消

### 环境变量传播
- 通过 `buildInheritedEnvVars()` 传播必要变量
- 包括 CLAUDECODE、实验标志、API 提供者变量

## 与其他文件的关系

### 依赖
- `../../utils/swarm/` — 团队管理、后端检测、布局管理
- `../../utils/task/framework.js` — 任务注册
- `../../utils/teammateMailbox.js` — 邮箱通信
- `../AgentTool/loadAgentsDir.js` — 自定义智能体定义

### 被依赖
- TeammateTool — 队友生成
- AgentTool — 智能体生成
