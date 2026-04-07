# prompts.ts — 会话记忆提示词模板

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/SessionMemory/prompts.ts`
- **作用域**: 会话记忆提取的提示词和模板
- **主要导出**:
  - `DEFAULT_SESSION_MEMORY_TEMPLATE`: 默认会话记忆模板
  - `loadSessionMemoryTemplate`: 加载自定义模板
  - `loadSessionMemoryPrompt`: 加载自定义提示词
  - `buildSessionMemoryUpdatePrompt`: 构建更新提示词
  - `isSessionMemoryEmpty`: 检查会话记忆是否为空
  - `truncateSessionMemoryForCompact`: 截断会话记忆用于压缩

## 功能概述

提供会话记忆的模板、提示词构建和内容处理功能。支持自定义模板和提示词，并提供节长度检查和截断功能。

## 核心内容详解

### 默认模板

```typescript
export const DEFAULT_SESSION_MEMORY_TEMPLATE = `
# Session Title
_A short and distinctive 5-10 word descriptive title for the session. Super info dense, no filler_

# Current State
_What is actively being worked on right now? Pending tasks not yet completed. Immediate next steps._

# Task specification
_What did the user ask to build? Any design decisions or other explanatory context_

# Files and Functions
_What are the important files? In short, what do they contain and why are they relevant?_

# Workflow
_What bash commands are usually run and in what order? How to interpret their output if not obvious?_

# Errors & Corrections
_Errors encountered and how they were fixed. What did the user correct? What approaches failed and should not be tried again?_

# Codebase and System Documentation
_What are the important system components? How do they work/fit together?_

# Learnings
_What has worked well? What has not? What to avoid? Do not duplicate items from other sections_

# Key results
_If the user asked a specific output such as an answer to a question, a table, or other document, repeat the exact result here_

# Worklog
_Step by step, what was attempted, done? Very terse summary for each step_
`
```

### 默认更新提示词

关键要点：
- **排除系统内容**: 不包括系统提示、CLAUDE.md 条目或过去会话摘要
- **仅使用 Edit 工具**: 更新指定文件后停止
- **并行编辑**: 单条消息中并行调用所有 Edit
- **结构保留**: 绝不修改、删除或添加章节标题，绝不修改或删除斜体描述行
- **仅更新内容**: 只更新斜体描述行下方的实际内容
- **详细内容**: 包含具体细节如文件路径、函数名、错误消息、确切命令
- **章节长度**: 每章节约 2000 tokens，超过时压缩
- **Current State**: 始终更新以反映最近工作

### 核心函数

#### `loadSessionMemoryTemplate()`
加载自定义模板：
- 路径：`~/.claude/session-memory/config/template.md`
- 不存在时返回默认模板

#### `loadSessionMemoryPrompt()`
加载自定义提示词：
- 路径：`~/.claude/session-memory/config/prompt.md`
- 支持 `{{variableName}}` 变量替换语法
- 可用变量：`{{currentNotes}}`、`{{notesPath}}`
- 不存在时返回默认提示词

#### `buildSessionMemoryUpdatePrompt(currentNotes, notesPath)`
构建完整的更新提示词：
1. 加载提示词模板
2. 分析章节大小
3. 生成章节长度提醒
4. 变量替换
5. 追加提醒

#### `isSessionMemoryEmpty(content)`
检查内容是否为空（仅包含模板）：
```typescript
return content.trim() === template.trim()
```

用于决定是使用会话记忆压缩还是回退到传统压缩。

#### `truncateSessionMemoryForCompact(content)`
截断会话记忆用于压缩：
- 每章节最大长度：2000 tokens
- 总长度限制：12000 tokens
- 在行边界处截断
- 添加 `[... section truncated for length ...]` 标记

### 限制常量

```typescript
const MAX_SECTION_LENGTH = 2000              // 每章节最大 tokens
const MAX_TOTAL_SESSION_MEMORY_TOKENS = 12000 // 总长度限制
```

### 辅助函数

#### `analyzeSectionSizes(content)`
分析各章节大小，返回章节标题到 token 数的映射

#### `generateSectionReminders(sectionSizes, totalTokens)`
生成章节长度提醒：
- 总长度超限：严重警告，必须压缩
- 单章节超限：重要提示，需要压缩

#### `substituteVariables(template, variables)`
单遍变量替换，支持 `{{variable}}` 语法

#### `flushSessionSection(sectionHeader, sectionLines, maxCharsPerSection)`
刷新章节内容，必要时截断

## 设计要点

1. **模板化结构**: 10 个标准章节覆盖会话关键信息
2. **可定制化**: 支持自定义模板和提示词
3. **长度保护**: 自动检测和提醒超长章节
4. **压缩友好**: 提供截断功能防止压缩后预算耗尽
5. **结构保护**: 强调保持章节标题和描述行的完整性

## 与其他文件的关系

- **sessionMemory.ts**: 调用 `buildSessionMemoryUpdatePrompt`
- **sessionMemoryCompact.ts**: 调用 `isSessionMemoryEmpty` 和 `truncateSessionMemoryForCompact`

## 注意事项

1. **自定义路径**:
   - 模板：`~/.claude/session-memory/config/template.md`
   - 提示词：`~/.claude/session-memory/config/prompt.md`

2. **结构重要性**: 章节标题和斜体描述行是模板结构，绝不能修改
3. **Current State 关键**: 此章节对压缩后连续性至关重要
4. **截断标记**: 截断时添加明确标记 `[... section truncated for length ...]`
