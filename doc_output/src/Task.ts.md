# Task.ts — 任务类型定义与生命周期管理

> **一句话总结**：定义任务的类型体系、状态机、唯一ID生成规则，以及任务状态对象的工厂函数。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/Task.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 125 |
| 主要职责 | 声明任务类型枚举、任务状态机枚举、任务相关接口，以及任务ID生成和初始状态对象创建的工厂函数。 |

---

## 功能概述

`Task.ts` 是整个任务调度子系统的基础类型层。它不包含任何业务逻辑实现，而是提供所有任务相关模块共享的类型契约与工厂工具函数。

该文件定义了系统支持的七种任务类型（本地Bash、本地Agent、远程Agent等），以及五种任务状态（pending/running/completed/failed/killed）。通过集中定义这些枚举和类型，避免了跨模块的类型分散问题。

任务ID采用"类型前缀 + 8位随机字符"的格式，字母表仅使用数字与小写字母，能够抵抗暴力枚举攻击（约2.8万亿种组合）。工厂函数 `createTaskStateBase` 为每种任务统一设置初始状态，确保所有任务都有统一的起始字段（如 `outputFile`、`outputOffset`、`notified`）。

---

## 核心内容详解

### 导入与依赖

| 模块 | 用途 |
|------|------|
| `crypto.randomBytes` | 生成加密随机字节，用于任务ID |
| `AppState` (类型) | 引用应用全局状态类型 |
| `AgentId` (类型) | Agent唯一标识类型 |
| `getTaskOutputPath` | 根据任务ID计算磁盘输出文件路径 |

### 主要类/函数/接口

- **名称**：`TaskType`
  - **类型**：union type
  - **用途**：枚举所有支持的任务类型
  - **取值**：`'local_bash'` | `'local_agent'` | `'remote_agent'` | `'in_process_teammate'` | `'local_workflow'` | `'monitor_mcp'` | `'dream'`

- **名称**：`TaskStatus`
  - **类型**：union type
  - **用途**：描述任务的生命周期状态
  - **取值**：`'pending'` | `'running'` | `'completed'` | `'failed'` | `'killed'`

- **名称**：`isTerminalTaskStatus`
  - **类型**：function
  - **用途**：判断任务是否处于终止状态（不会再转换）
  - **参数**：`status: TaskStatus`
  - **返回值**：`boolean`
  - **关键逻辑**：当 status 为 `'completed'`、`'failed'` 或 `'killed'` 时返回 `true`，用于防止向已死任务注入消息或进行孤儿清理

- **名称**：`TaskHandle`
  - **类型**：type（接口）
  - **用途**：持有任务ID和可选的清理回调
  - **字段**：`taskId: string`，`cleanup?: () => void`

- **名称**：`TaskContext`
  - **类型**：type（接口）
  - **用途**：任务执行上下文，包含中止控制器和状态访问器
  - **字段**：`abortController`、`getAppState`、`setAppState`

- **名称**：`TaskStateBase`
  - **类型**：type（接口）
  - **用途**：所有任务状态对象共有的基础字段集合
  - **字段**：`id`、`type`、`status`、`description`、`toolUseId?`、`startTime`、`endTime?`、`totalPausedMs?`、`outputFile`、`outputOffset`、`notified`

- **名称**：`LocalShellSpawnInput`
  - **类型**：type（接口）
  - **用途**：启动本地Shell任务的输入参数
  - **字段**：`command`、`description`、`timeout?`、`toolUseId?`、`agentId?`、`kind?`（`'bash'` | `'monitor'`）

- **名称**：`Task`
  - **类型**：type（接口）
  - **用途**：定义任务处理器的多态接口，仅保留 `kill` 方法（spawn/render已移除）
  - **字段**：`name`、`type`、`kill(taskId, setAppState): Promise<void>`

- **名称**：`generateTaskId`
  - **类型**：function
  - **用途**：为指定类型的任务生成唯一ID
  - **参数**：`type: TaskType`
  - **返回值**：`string`（格式：`{前缀}{8位随机字符}`）
  - **关键逻辑**：使用 `randomBytes(8)` 生成随机字节，从36字符字母表（`0-9a-z`）中选取字符，前缀由 `TASK_ID_PREFIXES` 映射表决定

- **名称**：`createTaskStateBase`
  - **类型**：function
  - **用途**：创建任务状态对象的初始基础字段
  - **参数**：`id`、`type`、`description`、`toolUseId?`
  - **返回值**：`TaskStateBase`
  - **关键逻辑**：状态初始为 `'pending'`，`startTime` 取当前时间，`outputFile` 由 `getTaskOutputPath(id)` 计算，`outputOffset` 为 0，`notified` 为 `false`

### 数据流与逻辑流程

```
调用方（如 BashTool）
  → generateTaskId(type)        // 生成唯一ID
  → createTaskStateBase(...)    // 创建初始状态
  → 存入 AppState.tasks
  → 状态流转：pending → running → completed/failed/killed
  → isTerminalTaskStatus() 检查是否可以清理
```

### 对外接口（导出内容）

全部内容均为导出：`TaskType`、`TaskStatus`、`isTerminalTaskStatus`、`TaskHandle`、`SetAppState`、`TaskContext`、`TaskStateBase`、`LocalShellSpawnInput`、`Task`、`generateTaskId`、`createTaskStateBase`。

---

## 设计要点

- **前缀区分类型**：任务ID前缀（`b`/`a`/`r`/`t`/`w`/`m`/`d`）便于在日志和调试中快速识别任务类型，`b` 保持向后兼容。
- **安全随机性**：使用 `crypto.randomBytes` 而非 `Math.random()`，防止预测任务ID进行符号链接攻击。
- **最小多态接口**：`Task` 接口仅保留 `kill` 方法，注释明确说明 `spawn`/`render` 已被删除（PR #22546），体现了接口的单一职责。
- **工厂函数统一初始化**：`createTaskStateBase` 避免各模块重复初始化任务状态，防止遗漏字段。

---

## 与其他文件的关系

- **依赖**：
  - `src/state/AppState.ts`（AppState 类型）
  - `src/types/ids.ts`（AgentId 类型）
  - `src/utils/task/diskOutput.ts`（getTaskOutputPath）
- **被依赖**：
  - 各任务实现模块（BashTool、AgentTool、WorkflowTool 等）
  - `src/QueryEngine.ts`（TaskContext 类型）
  - `src/state/AppState.ts`（任务状态存储）

---

## 注意事项

- `Task` 接口的 `kill` 方法签名只接收 `setAppState`，不含 `getAppState` 和 `abortController`（注释中说明这些是"dead weight"已删除）；任务实现若需要访问状态须通过闭包捕获。
- `TASK_ID_PREFIXES` 使用 `Record<string, string>` 而非类型安全的 `Record<TaskType, string>`，因此新增任务类型时需手动维护映射表，否则会退回到默认前缀 `'x'`。
- `totalPausedMs` 和 `endTime` 为可选字段，计算任务实际耗时时需注意处理 `undefined`。
