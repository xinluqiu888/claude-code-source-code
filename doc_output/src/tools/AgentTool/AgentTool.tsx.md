# AgentTool.tsx — Agent工具核心实现，负责子代理的启动与生命周期管理

> **一句话总结**：AgentTool 是 Claude Code 中最核心的工具之一，负责启动和管理各类子代理（subagent），支持同步/异步执行、工作树隔离、远程执行及多代理协作等模式。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/tools/AgentTool/AgentTool.tsx` |
| 文件类型 | TSX |
| 代码行数 | 约850行（文件极大，超出单次读取限制） |
| 主要职责 | 定义 AgentTool（原 Task Tool）的输入/输出 schema、执行逻辑、权限检查和多种代理启动模式 |

---

## 功能概述

AgentTool 是 Claude Code 多代理系统的调度中枢，通过该工具，主代理（parent agent）可以将子任务委派给各种专门化的子代理（subagent）来并行或串行执行。

工具名称为 `Agent`（旧名称 `Task`，保留向后兼容），支持以下核心能力：
1. **同步执行**：直接运行子代理，等待其完成并返回结果
2. **异步（后台）执行**：通过 `run_in_background: true` 在后台启动代理，立即返回任务 ID 和输出文件路径
3. **工作树隔离**：通过 `isolation: "worktree"` 创建独立的 git worktree，子代理在隔离环境中工作
4. **远程执行**：通过 `isolation: "remote"` 在远程 CCR 环境中执行（仅限内部构建）
5. **多代理协作（Agent Swarms）**：通过 `name` 参数将子代理注册为有名称的队友（teammate）

在系统架构中，AgentTool 处于顶层调度位置，协调所有子代理的创建、权限管控、工具分配和生命周期管理。

---

## 核心内容详解

### 导入与依赖

- `zod/v4`：输入/输出 schema 定义与运行时校验
- `buildTool`、`ToolDef`：工具框架基础
- `runAgent`：子代理实际执行逻辑（来自 `./runAgent.ts`）
- `LocalAgentTask`：本地异步代理任务注册与管理
- `RemoteAgentTask`：远程代理任务注册
- `spawnTeammate`：多代理模式下生成队友
- `filterDeniedAgents`：权限过滤
- `assembleToolPool`：子代理工具池组装

### 主要类/函数/接口

#### 输入 Schema（`inputSchema`）
```typescript
{
  description: string,        // 任务描述（3-5词）
  prompt: string,             // 代理执行的任务提示
  subagent_type?: string,     // 指定代理类型（如 "Explore", "Plan" 等）
  model?: 'sonnet'|'opus'|'haiku',  // 可选模型覆盖
  run_in_background?: boolean,       // 是否后台运行
  name?: string,              // 多代理模式下的代理名称（KAIROS功能门控）
  team_name?: string,         // 队伍名称（KAIROS功能门控）
  mode?: PermissionMode,      // 权限模式
  isolation?: 'worktree'|'remote',  // 隔离模式
  cwd?: string,               // 工作目录覆盖（KAIROS功能门控）
}
```

#### 输出 Schema（`outputSchema`）
两种状态的联合类型：
- `status: 'completed'`：同步完成，包含结果和 prompt
- `status: 'async_launched'`：异步启动，包含 agentId、outputFile 等

内部还有：
- `status: 'teammate_spawned'`：多代理协作模式（不导出）
- `status: 'remote_launched'`：远程启动（导出用于 UI 类型推断）

#### `call()` 方法主要流程
1. 检查 Agent Swarms 功能门控和 team 权限
2. 若有 `name` 且 `teamName`，调用 `spawnTeammate()` 进入多代理路径
3. 否则解析 `effectiveType`（显式指定 or fork 路径）
4. 过滤被权限规则拒绝的代理
5. 检查所需 MCP 服务器是否已连接并认证
6. 初始化代理颜色（UI 显示用）
7. 根据 `effectiveIsolation` 处理远程或本地隔离
8. 构建系统提示和消息（fork 路径继承父代理系统提示）
9. 注册后台任务（若异步）或直接调用 `runAgent()`

### 数据流与逻辑流程

```
用户请求 → AgentTool.call()
  → 权限检查（filterDeniedAgents）
  → 代理类型解析（subagent_type / fork）
  → MCP 服务器可用性检查
  → 隔离模式决策
     ├─ remote → teleportToRemote() → registerRemoteAgentTask()
     ├─ worktree → createAgentWorktree() → runAgent()
     └─ none → runAgent()
  → 同步/异步模式
     ├─ async → registerAsyncAgent() → runAsyncAgentLifecycle()（后台）
     └─ sync → 直接等待 runAgent() 结果
```

### 对外接口（导出内容）
- `inputSchema`：工具输入 schema（lazySchema 形式）
- `outputSchema`：工具输出 schema
- `AgentTool`：使用 `buildTool()` 构建的工具对象
- `RemoteLaunchedOutput`：远程启动结果类型
- `Progress`：进度事件类型联合（`AgentToolProgress | ShellProgress`）

---

## 设计要点

### 安全性与权限模型
- 通过 `filterDeniedAgents()` 实现代理级别的权限控制，支持 `Agent(AgentName)` 语法的细粒度拒绝规则
- 子代理的权限模式可独立设置（`mode` 参数），支持 `plan`（需审批）等模式
- 原地守卫：Fork 子代理不能再发起 Fork（递归保护），通过 `querySource` 和消息扫描双重检测
- 队友（teammate）不能再生成其他队友，防止嵌套团队问题

### 功能门控（Feature Gates）
多处代码通过 `feature('FEATURE_NAME')` 或 GrowthBook 进行特性控制：
- `KAIROS`：启用 `cwd`、`name`、`team_name`、`mode` 参数
- `FORK_SUBAGENT`：启用 Fork 路径（子代理继承父代理完整上下文）
- `COORDINATOR_MODE`：协调器模式特殊处理
- 后台任务：通过 `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` 环境变量或 GrowthBook 控制

### 错误处理
- MCP 服务器等待：若所需 MCP 服务器正在连接中，最多等待 30 秒
- 远程执行失败提供 `bundleFailHint` 错误提示
- 代理类型不存在时，区分"被拒绝"和"不存在"提供不同错误信息

---

## 与其他文件的关系

- **依赖**：
  - `./runAgent.ts`：子代理实际执行
  - `./agentToolUtils.ts`：工具过滤、进度发射、生命周期管理工具函数
  - `./forkSubagent.ts`：Fork 子代理功能
  - `./builtInAgents.ts` / `./loadAgentsDir.ts`：代理定义加载
  - `../../tasks/LocalAgentTask/LocalAgentTask.ts`：异步任务管理
  - `../../tasks/RemoteAgentTask/RemoteAgentTask.ts`：远程任务管理
  - `../../tools/shared/spawnMultiAgent.ts`：多代理生成
  - `./agentColorManager.ts`：代理颜色管理
  - `./UI.js`：UI 渲染函数

- **被依赖**：
  - 顶层工具注册（`tools.ts`）
  - 多处测试文件
  - `./resumeAgent.ts`（共享 `runAgent` 调用模式）

---

## 注意事项

- 队友（`name` 参数）只能在 `KAIROS` 功能开启时使用
- 在进程内队友（in-process teammate）中，无法启动后台代理
- `run_in_background` 参数在 `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` 时从 schema 中移除，模型不可见
- 自动后台化：若 `CLAUDE_AUTO_BACKGROUND_TASKS=true` 或 GrowthBook 开启，代理在 120 秒后自动转后台
- Fork 路径的子代理继承父代理系统提示（byte-exact），以确保 prompt cache 命中
