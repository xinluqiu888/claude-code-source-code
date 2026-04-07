# systemInit.ts — SDK系统初始化消息构建

> **一句话总结**：构建system/init SDK消息，携带会话元数据（工具、模型、命令等）。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/messages/systemInit.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~97行 |
| 主要职责 | 构建SDK初始化消息 |

---

## 功能概述

该模块构建`system/init`类型的SDKMessage，这是SDK流的第一条消息，包含：
- 会话基本信息（cwd、session_id）
- 可用工具和MCP服务器
- 当前模型和权限模式
- 可用的斜杠命令和agents
- API密钥来源和beta功能

该消息被两个路径调用：
1. QueryEngine（spawn-bridge/print-mode/SDK）
2. useReplBridge（REPL Remote Control）

---

## 核心内容详解

### 导出类型

- **`SystemInitInputs`**
  - tools: 可用工具列表
  - mcpClients: MCP客户端列表
  - model: 当前模型
  - permissionMode: 权限模式
  - commands: 斜杠命令列表
  - agents: Agent类型列表
  - skills: 技能列表
  - plugins: 插件列表
  - fastMode: 快速模式状态

### 导出函数

- **`buildSystemInitMessage`**
  - 参数：`inputs`（SystemInitInputs）
  - 返回值：`SDKMessage`
  - 构建内容：
    - 基本信息：cwd, session_id
    - 工具：tools, mcp_servers
    - 配置：model, permissionMode, output_style
    - 功能：slash_commands, agents, skills
    - 来源：apiKeySource, betas, claude_code_version
    - 扩展：fast_mode_state, messaging_socket_path（UDS特性）

- **`sdkCompatToolName`**
  - 用途：兼容性处理，将'Agent'工具名映射到'Task'（遗留命名）

---

## 与其他文件的关系

- **依赖**：
  - `src/bootstrap/state.ts` - sessionId, betas
  - `../auth.ts` - API密钥来源
  - `../cwd.ts` - 当前目录
  - `../fastMode.ts` - 快速模式状态
- **被依赖**：
  - QueryEngine
  - REPL bridge

---

## 注意事项

- UDS_INBOX特性会添加messaging_socket_path（ant-only）
- TODO注释：'Task'命名是遗留，计划迁移到'Agent'
- 消息用于远程客户端渲染选择器和UI
