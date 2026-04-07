# coordinatorMode.ts — 协调器模式管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/coordinator/coordinatorMode.ts`
- **类型**: TypeScript 模块
- **语言**: TypeScript

## 功能概述

管理 Claude Code 的协调器模式 (Coordinator Mode)，该模式支持多工作流并行处理。协调器负责任务分解、工作流调度和结果合成，而实际工作由子工作流执行。

## 核心内容详解

### 主要导出

1. **isCoordinatorMode()** — 检查是否处于协调器模式
   - 检查 `CLAUDE_CODE_COORDINATOR_MODE` 环境变量
   - 仅在 `COORDINATOR_MODE` 特性启用时生效

2. **matchSessionMode(sessionMode)** — 匹配会话模式
   - 检查当前协调器模式是否与会话的存储模式匹配
   - 如果不匹配，翻转环境变量以匹配恢复会话
   - 返回警告消息或 undefined

3. **getCoordinatorUserContext(mcpClients, scratchpadDir?)** — 获取协调器用户上下文
   - 返回工作流可用工具的上下文信息
   - 包含 MCP 服务器信息和便签本目录

4. **getCoordinatorSystemPrompt()** — 获取协调器系统提示
   - 返回完整的协调器模式系统提示词
   - 包含角色定义、工具说明、工作流指导等

### 常量

```typescript
const INTERNAL_WORKER_TOOLS = new Set([
  TEAM_CREATE_TOOL_NAME,
  TEAM_DELETE_TOOL_NAME,
  SEND_MESSAGE_TOOL_NAME,
  SYNTHETIC_OUTPUT_TOOL_NAME,
])
```

### 工作流可用工具

- **简化模式**: Bash, Read, Edit
- **标准模式**: 所有异步工作流允许的工具 (排除内部工作流工具)

## 设计要点

1. **特性门控**:
   - 使用 `feature('COORDINATOR_MODE')` 控制功能可用性
   - 通过 GrowthBook 进行动态特性开关

2. **会话模式匹配**:
   - 支持恢复协调器模式会话
   - 自动切换环境变量以匹配会话状态
   - 记录模式切换事件

3. **工具上下文注入**:
   - 动态生成工作流可用工具列表
   - 包含 MCP 服务器信息
   - 支持便签本目录 (Scratchpad)

4. **系统提示词**:
   - 详细的协调器角色定义
   - 工作流任务分配策略
   - 并行处理指导
   - 结果合成指南

## 与其他文件的关系

- **../constants/tools.js**: ASYNC_AGENT_ALLOWED_TOOLS 常量
- **../tools/**: 各种工具名称常量
- **../services/analytics/growthbook.js**: 特性门控检查

## 注意事项

1. 协调器模式需要显式启用特性门控
2. 环境变量 `CLAUDE_CODE_COORDINATOR_MODE` 控制模式状态
3. 会话恢复时自动匹配模式
4. 便签本功能需要 `tengu_scratch` 特性门控
5. 系统提示词包含完整的工作流协调指导
