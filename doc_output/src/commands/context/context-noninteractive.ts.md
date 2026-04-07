# context-noninteractive.ts — 上下文使用非交互式输出

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/context/context-noninteractive.ts` |
| 文件类型 | TypeScript (.ts) |
| 主要职责 | 为非交互式会话提供上下文使用信息（Markdown 表格格式） |

## 功能概述

该文件实现了 `/context` 命令的非交互式版本，用于 CI/CD 或非交互式会话。它以 Markdown 表格格式输出上下文使用信息，而不是彩色可视化。

## 核心内容详解

### 主要函数

**collectContextData**
```typescript
export async function collectContextData(
  context: CollectContextDataInput,
): Promise<ContextData>
```
- 收集上下文数据供 SDK 控制请求使用
- 应用与 query.ts 相同的预处理（compact boundary、projectView、microcompact）

**call 函数**
```typescript
export async function call(
  _args: string,
  context: ToolUseContext,
): Promise<{ type: 'text'; value: string }>
```
- 收集上下文数据
- 格式化为 Markdown 表格
- 返回文本响应

### 输出内容

**formatContextAsMarkdownTable** 函数生成的表格包括：

1. **头部信息**：
   - 模型名称
   - Token 使用情况（当前/最大/百分比）
   - Context Collapse 状态（如果启用）

2. **类别表格**：
   - 各类别的 token 计数和百分比
   - Free space 和 Autocompact buffer

3. **详细列表**：
   - MCP Tools
   - System Tools（仅 ant）
   - System Prompt Sections（仅 ant）
   - Custom Agents
   - Memory Files
   - Skills
   - Message Breakdown（仅 ant）

## 设计要点

1. **Markdown 格式**：适合非交互式环境阅读
2. **条件输出**：根据特性标志和用户类型输出不同内容
3. **完整数据**：比交互式版本提供更详细的信息
4. **SDK 兼容**：也可用于 SDK 控制请求

## 与其他文件的关系

- **context.tsx**: 交互式版本
- **analyzeContext.ts**: 提供分析功能
- **index.ts**: 命令注册
