# toolOrchestration.ts — 工具编排

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/tools/toolOrchestration.ts`
- **所属模块**: Tools Service
- **功能类型**: 工具执行编排

## 功能概述

编排多个工具的执行，支持并发和串行执行模式，基于工具的并发安全性自动分区。

## 核心内容详解

### 常量

```typescript
// 最大工具使用并发数
function getMaxToolUseConcurrency(): number {
  return parseInt(process.env.CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY || '', 10) || 10
}
```

### 分区策略

#### `partitionToolCalls(toolUseMessages, toolUseContext): Batch[]`
将工具调用分区为批次。

**批次类型：**
1. **单非只读工具** — 串行执行
2. **多连续只读工具** — 并发执行

**判断逻辑：**
- 查找工具定义
- 解析输入
- 调用 `tool.isConcurrencySafe(parsedInput)`
- 相同类型连续合并

### 执行模式

#### `runToolsSerially(blocks, ...)`
串行执行工具。

**流程：**
1. 设置进行中工具 ID
2. 运行工具
3. 应用上下文修改器
4. 标记完成

#### `runToolsConcurrently(blocks, ...)`
并发执行工具。

**流程：**
1. 使用 `all()` 并发执行
2. 限制并发数（默认 10）
3. 每个工具独立追踪

### 主函数

#### `runTools(toolUseMessages, assistantMessages, canUseTool, toolUseContext)`
编排工具执行。

**流程：**
1. 分区工具调用
2. 遍历批次：
   - 只读批次 → 并发执行
   - 非只读批次 → 串行执行
3. 应用上下文修改器
4. 更新进行中工具 ID
5. 生成消息更新

### 上下文修改器

#### `MessageUpdateLazy`
```typescript
{
  message?: Message
  contextModifier?: {
    toolUseID: string
    modifyContext: (context: ToolUseContext) => ToolUseContext
  }
}
```

**用途：**
- 延迟应用上下文修改
- 并发执行时保持顺序

## 设计要点

1. **自动分区** — 基于 `isConcurrencySafe` 自动优化
2. **并发控制** — 限制最大并发数
3. **上下文一致性** — 延迟修改器保证顺序
4. **进行中追踪** — `setInProgressToolUseIDs`
5. **生成器模式** — 流式产出结果

## 与其他文件的关系

- **依赖**: `toolExecution.ts` 的 `runToolUse`
- **关联**: `all.ts` 并发工具

## 注意事项

- 只读工具并发安全，非只读工具串行执行
- 环境变量可覆盖默认并发数
- 上下文修改器延迟到批次结束应用
