# coreSchemas.ts — SDK 核心 Schema

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/entrypoints/sdk/coreSchemas.ts`
- **类型**: TypeScript Schema 定义
- **语言**: TypeScript
- **文件大小**: 较大 (超过10000 tokens)

## 功能概述

SDK 核心的 Zod schema 定义，包含消息、配置、工具等可序列化类型的运行时验证 schema。这些 schema 是类型的来源，但本身不是公共 API。

## 核心内容分类

### 基础类型 Schema

1. **ModelInfoSchema** — 模型信息
2. **AccountInfoSchema** — 账户信息
3. **FastModeStateSchema** — 快速模式状态
4. **PermissionModeSchema** — 权限模式
5. **PermissionUpdateSchema** — 权限更新

### 消息类型 Schema

1. **SDKUserMessageSchema** — 用户消息
2. **SDKAssistantMessageSchema** — 助手消息
3. **SDKToolResultMessageSchema** — 工具结果消息
4. **SDKMessageSchema** — 通用消息联合

### 工具相关 Schema

1. **ToolInputSchema** — 工具输入
2. **ToolOutputSchema** — 工具输出
3. **ToolAnnotationsSchema** — 工具注解
4. **ToolErrorSchema** — 工具错误

### MCP 相关 Schema

1. **McpServerConfigSchema** — MCP 服务器配置
2. **McpServerConfigForProcessTransportSchema** — 进程传输配置
3. **McpServerStatusSchema** — MCP 服务器状态
4. **McpToolSchema** — MCP 工具

### Agent 相关 Schema

1. **AgentDefinitionSchema** — 工作流定义
2. **AgentInfoSchema** — 工作流信息
3. **SubagentStartSchema** — 子工作流启动
4. **SubagentStopSchema** — 子工作流停止

### Hook 相关 Schema

1. **HookEventSchema** — Hook 事件
2. **HookInputSchema** — Hook 输入

### 斜杠命令 Schema

**SlashCommandSchema** — 斜杠命令定义

### 流式消息 Schema

1. **SDKStreamlinedTextMessageSchema** — 简化文本消息
2. **SDKStreamlinedToolUseSummaryMessageSchema** — 简化工具使用摘要
3. **SDKPostTurnSummaryMessageSchema** — 回合后摘要

## 设计要点

1. **延迟 Schema**:
   - 所有 schema 使用 `lazySchema` 包装
   - 避免模块加载时的循环依赖
   - 支持按需创建

2. **描述性文档**:
   - 每个 schema 都有详细描述
   - 使用 `.describe()` 方法

3. **可选字段**:
   - 可选字段使用 `.optional()`
   - 合理的默认值

4. **联合类型**:
   - 使用 `z.union` 定义多种可能类型
   - 使用 `z.discriminatedUnion` 提高性能

5. **记录类型**:
   - 使用 `z.record` 定义动态键对象
   - 用于 hooks、工具集合等

## 类型生成

这些 schema 用于生成 coreTypes.generated.ts 中的类型：

```bash
bun scripts/generate-sdk-types.ts
```

## 与其他文件的关系

- **./coreTypes.ts**: 重新导出生成的类型
- **./coreTypes.generated.ts**: 生成的类型文件
- **./controlSchemas.ts**: 导入部分基础 schema
- **../../utils/lazySchema.ts**: lazySchema 实现

## 注意事项

1. 这是类型的来源，但不是公共 API
2. SDK 消费者应使用 coreTypes.ts
3. 修改 schema 后需要重新生成类型
4. 所有 schema 都是延迟创建的
5. 联合类型使用 z.union 或 z.discriminatedUnion
