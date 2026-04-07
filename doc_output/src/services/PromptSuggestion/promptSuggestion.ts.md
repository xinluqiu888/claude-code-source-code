# promptSuggestion.ts — 提示建议生成

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/PromptSuggestion/promptSuggestion.ts`
- **作用域**: 提示建议生成和过滤
- **主要导出**:
  - `shouldEnablePromptSuggestion`: 检查是否启用提示建议
  - `executePromptSuggestion`: 执行提示建议生成
  - `tryGenerateSuggestion`: 尝试生成建议
  - `generateSuggestion`: 生成建议的核心函数
  - `shouldFilterSuggestion`: 建议过滤
  - `getSuggestionSuppressReason`: 获取抑制原因
  - `logSuggestionOutcome`: 记录建议结果

## 功能概述

根据对话上下文预测用户接下来可能输入的内容，并生成提示建议。使用 forked agent 模式在主对话的缓存上运行，避免额外的 cache creation 成本。

## 核心内容详解

### 启用检查

#### `shouldEnablePromptSuggestion()`
检查是否启用提示建议：
1. **环境变量覆盖**: `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION`
2. **特性标志**: `tengu_chomp_inflection`
3. **非交互模式**: 禁用（print 模式、管道输入、SDK）
4. **Swarm 队友**: 禁用（仅 leader 显示建议）
5. **用户设置**: `promptSuggestionEnabled`

### 建议生成

#### `tryGenerateSuggestion(abortController, messages, getAppState, cacheSafeParams, source?)`

完整的建议生成流程：
1. 检查中止信号
2. 检查助手轮次数量（至少 2 轮）
3. 检查最后助手消息是否为 API 错误
4. 检查父缓存状态（冷缓存抑制）
5. 检查应用状态抑制条件
6. 调用 `generateSuggestion`
7. 过滤建议

#### `generateSuggestion(abortController, promptId, cacheSafeParams)`

核心建议生成：
- **提示词**: `SUGGESTION_PROMPT`
- **模式**: 预测用户自然输入的内容
- **工具**: 通过 `canUseTool` 回调拒绝所有工具（不传 `tools:[]` 以避免破坏缓存）
- **缓存复用**: 复用主线程的 prompt cache

**提示词要点**:
- 关注用户最近消息和原始请求
- 预测用户会输入什么（不是应该输入什么）
- 测试："他们会想'我正要打那个'吗？"
- 格式：2-12 个词，匹配用户风格，或为空

### 建议过滤

#### `shouldFilterSuggestion(suggestion, promptId, source?)`

多层过滤器：

| 过滤器 | 条件 |
|--------|------|
| `done` | 内容为 "done" |
| `meta_text` | "nothing found"、"stay silent" 等元文本 |
| `meta_wrapped` | 被括号/方括号包裹的元推理 |
| `error_message` | API 错误消息前缀 |
| `prefixed_label` | "Label: text" 格式 |
| `too_few_words` | 少于 2 个词（允许的单词除外） |
| `too_many_words` | 超过 12 个词 |
| `too_long` | 超过 100 字符 |
| `multiple_sentences` | 多句话 |
| `has_formatting` | 包含换行或 Markdown |
| `evaluative` | "thanks"、"looks good" 等评价性语言 |
| `claude_voice` | Claude 风格（"Let me..."、"I'll..."） |

允许的单字命令：yes, yeah, yep, yea, yup, sure, ok, okay, push, commit, deploy, stop, continue, check, exit, quit, no

### 抑制条件

#### `getSuggestionSuppressReason(appState)`

返回抑制原因或 null：
- `disabled`: 功能禁用
- `pending_permission`: 有待处理的权限请求
- `elicitation_active`: 正在获取更多信息
- `plan_mode`: 处于计划模式
- `rate_limit`: 外部用户速率限制

### 缓存抑制

#### `getParentCacheSuppressReason(lastAssistantMessage)`

如果父请求未缓存 tokens 超过 10,000，则抑制建议生成（避免冷缓存成本）。

### 遥测

#### `logSuggestionOutcome(suggestion, userInput, emittedAt, promptId, generationRequestId)`
记录建议接受/忽略结果：
- 计算相似度
- 记录时间
- 记录来源（SDK/CLI）

#### `logSuggestionSuppressed(reason, suggestion?, promptId?, source?)`
记录建议被抑制的原因

## 设计要点

1. **缓存复用**: 通过不传 `tools:[]` 复用主线程的 prompt cache
2. **工具拒绝**: 通过 `canUseTool` 回调拒绝工具，避免破坏缓存键
3. **多层过滤**: 12 层过滤器确保建议质量
4. **用户意图**: 提示词专注于预测用户自然输入，而非最优操作
5. **遥测完整**: 记录启用、抑制、过滤、接受/忽略全链路

## 与其他文件的关系

- **speculation.ts**: 集成推测执行功能
- **forkedAgent.ts**: 提供 forked agent 执行
- **growthbook.ts**: 特性标志检查
- **claudeAiLimits.ts**: 速率限制检查

## 注意事项

1. **缓存敏感**: 不要覆盖任何影响缓存键的参数
2. **环境变量**: `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` 可强制启用/禁用
3. **Feature Flag**: `tengu_chomp_inflection` 控制功能可见性
4. **Ant 专用**: 详细建议内容仅对 ant 用户记录
