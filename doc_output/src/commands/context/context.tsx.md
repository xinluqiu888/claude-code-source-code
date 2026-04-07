# context.tsx — 上下文使用可视化

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/context/context.tsx` |
| 文件类型 | TypeScript React (.tsx) |
| 主要职责 | 提供当前上下文使用的彩色网格可视化 |

## 功能概述

该文件实现了 `/context` 命令的交互式版本，用于可视化当前会话的上下文使用情况。它以彩色网格的形式展示 token 分布，帮助用户理解哪些内容占用了上下文空间。

## 核心内容详解

### 主要函数

**toApiView**
```typescript
function toApiView(messages: Message[]): Message[]
```
- 应用与 query.ts 相同的上下文转换
- 包括 compact boundary 过滤和 context collapse
- 确保显示的 token 计数与 API 实际接收的一致

**call 函数**
```typescript
export async function call(
  onDone: LocalJSXCommandOnDone,
  context: LocalJSXCommandContext,
): Promise<React.ReactNode>
```

### 执行流程

1. **获取 API 视图**：`toApiView(messages)` 转换消息
2. **微压缩**：`microcompactMessages(apiView)` 获取准确表示
3. **获取终端宽度**：`process.stdout.columns || 80`
4. **分析上下文**：`analyzeContextUsage(...)` 生成分析数据
5. **渲染**：`renderToAnsiString(<ContextVisualization data={data} />)`
6. **输出**：`onDone(output)` 返回 ANSI 字符串

### ContextVisualization 组件

- 显示彩色网格表示 token 分布
- 包含各类别的 token 计数
- 支持响应式布局（基于终端宽度）

## 设计要点

1. **一致性**：使用与 API 调用相同的预处理逻辑
2. **彩色输出**：使用 ANSI 颜色代码
3. **静态渲染**：渲染为 ANSI 字符串而非交互式组件
4. **响应式**：根据终端宽度调整布局

## 与其他文件的关系

- **context-noninteractive.ts**: 非交互式版本的实现
- **analyzeContext.ts**: 提供 `analyzeContextUsage` 函数
- **microCompact.ts**: 提供 `microcompactMessages` 函数
- **ContextVisualization.tsx**: 可视化组件
