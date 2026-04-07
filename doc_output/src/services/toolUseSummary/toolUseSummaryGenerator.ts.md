# toolUseSummaryGenerator.ts — 工具使用摘要生成器

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/toolUseSummary/toolUseSummaryGenerator.ts`
- **作用域**: 生成工具执行摘要（使用 Haiku 模型）
- **主要导出**:
  - `generateToolUseSummary`: 生成工具使用摘要
  - `GenerateToolUseSummaryParams`: 参数类型

## 功能概述

使用 Claude Haiku 模型为已完成的工具调用批次生成人类可读的摘要。摘要以单行标签形式显示在移动应用中，约30字符截断，类似 git commit 主题风格。

## 核心内容详解

### 类型定义

#### `ToolInfo`
工具信息结构：
```typescript
type ToolInfo = {
  name: string      // 工具名称
  input: unknown    // 工具输入
  output: unknown   // 工具输出
}
```

#### `GenerateToolUseSummaryParams`
生成摘要参数：
```typescript
export type GenerateToolUseSummaryParams = {
  tools: ToolInfo[]              // 工具信息数组
  signal: AbortSignal            // 中止信号
  isNonInteractiveSession: boolean  // 是否为非交互式会话
  lastAssistantText?: string     // 助手最后一条消息（可选）
}
```

### 系统提示词

```
Write a short summary label describing what these tool calls accomplished. 
It appears as a single-line row in a mobile app and truncates around 30 characters, 
so think git-commit-subject, not sentence.

Keep the verb in past tense and the most distinctive noun. 
Drop articles, connectors, and long location context first.

Examples:
- Searched in auth/
- Fixed NPE in UserService
- Created signup endpoint
- Read config.json
- Ran failing tests
```

**提示词要点**:
- 单行显示，约30字符截断
- 使用过去时态
- 保留最具代表性的名词
- 省略冠词、连接词和长路径

### 核心函数

#### `generateToolUseSummary(params)`
生成工具使用摘要：

**参数**:
- `tools`: 已执行的工具列表
- `signal`: AbortSignal 用于取消
- `isNonInteractiveSession`: 是否非交互会话
- `lastAssistantText`: 助手最后消息（用于上下文）

**流程**:
1. 空工具列表检查
2. 构建工具摘要：
   - 截断 input 和 output 至300字符
   - 格式: `Tool: {name}\nInput: {input}\nOutput: {output}`
3. 添加上下文前缀（如果有 lastAssistantText）
4. 调用 `queryHaiku` 生成摘要
5. 提取并清理响应文本
6. 返回摘要或 null（失败时）

**错误处理**:
- 失败时记录错误（`E_TOOL_USE_SUMMARY_GENERATION_FAILED`）
- 返回 null 而非抛出，摘要为非关键功能

#### `truncateJson(value, maxLength)`
截断 JSON 值：

**逻辑**:
1. 使用 `jsonStringify` 序列化
2. 如果超过最大长度，截断并添加 `...`
3. 序列化失败返回 `[unable to serialize]`

### 示例输出

- "Searched in auth/"
- "Fixed NPE in UserService"
- "Created signup endpoint"
- "Read config.json"
- "Ran failing tests"

## 设计要点

1. **简洁性**: 单行摘要，约30字符截断
2. **过去时态**: 动词使用过去式
3. **容错性**: 失败时返回 null 不中断流程
4. **上下文感知**: 使用助手最后消息提供用户意图上下文
5. **截断处理**: input/output 截断至300字符控制提示长度

## 与其他文件的关系

- **services/api/claude.ts**: 提供 `queryHaiku`
- **utils/errors.ts**: 提供 `toError`
- **utils/log.ts**: 提供 `logError`
- **utils/slowOperations.ts**: 提供 `jsonStringify`
- **utils/systemPromptType.ts**: 提供 `asSystemPrompt`
- **constants/errorIds.ts**: 错误ID定义

## 注意事项

1. **非关键功能**: 摘要生成失败不影响主流程
2. **Haiku 模型**: 使用轻量级模型控制成本
3. **移动优先**: 设计考虑移动应用单行显示
4. **提示缓存**: 启用 `enablePromptCaching: true`
5. ** agents/MCP**: 传空数组，摘要生成不使用外部工具
