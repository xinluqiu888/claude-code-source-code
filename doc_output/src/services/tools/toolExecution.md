# toolExecution.ts — 工具执行引擎

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/tools/toolExecution.ts`
- **所属模块**: Tools Service
- **功能类型**: 工具执行核心

## 功能概述

工具执行的核心引擎，处理工具调用生命周期、权限检查、钩子执行和结果处理。

## 核心内容详解

### 错误分类

#### `classifyToolError(error): string`
将工具错误分类为遥测安全字符串。

**策略：**
- `TelemetrySafeError` — 使用预验证消息
- Node.js fs 错误 — 使用错误代码（ENOENT, EACCES）
- 已知类型 — 使用 `.name`
- 默认 — `"Error"`

### 权限决策映射

#### `ruleSourceToOTelSource(ruleSource, behavior)`
将规则来源映射到 OTel source 标签。

| 来源 | Allow | Deny |
|------|-------|------|
| session | user_temporary | user_reject |
| localSettings/userSettings | user_permanent | user_reject |
| 其他 | config | config |

### 工具执行流程

1. **权限检查** — 运行前钩子决策
2. **工具查找** — 根据名称找到工具实现
3. **执行** — 调用工具实现
4. **后处理** — 运行后钩子
5. **结果格式化** — 转换为 API 格式
6. **遥测** — 记录分析事件

### MCP 工具处理

- `findMcpServerConnection` — 查找 MCP 服务器
- `mcpInfoFromString` / `normalizeNameForMCP` — 名称解析

### 分析事件

- 工具调用成功/失败
- MCP 工具详情（服务器名、工具名）
- 工具输入内容（遥测安全）

### 会话活动

- `startSessionActivity` — 工具执行开始
- `stopSessionActivity` — 工具执行结束

## 设计要点

1. **错误安全** — 避免在遥测中暴露代码/路径
2. **权限追踪** — 详细记录决策来源
3. **MCP 集成** — 无缝支持 MCP 工具
4. **钩子系统** — 可扩展的前/后处理
5. **会话跟踪** — 活动状态管理

## 与其他文件的关系

- **被调用**: 查询引擎
- **依赖**: `toolHooks.ts`, `../mcp/client.ts`
- **关联**: 分析系统、遥测系统

## 注意事项

- 工具名称 `mcp__` 前缀标识 MCP 工具
- 权限决策来源影响 OTel 标签
- 错误分类防止 minified 名称泄露
