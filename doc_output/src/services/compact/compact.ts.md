# compact.ts — 对话压缩与摘要生成核心模块

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/compact/compact.ts`
- **作用域**: 对话上下文管理、自动/手动压缩、会话摘要生成
- **主要导出**:
  - `compactConversation`: 完整对话压缩函数
  - `partialCompactConversation`: 部分对话压缩函数
  - `CompactionResult`: 压缩结果类型定义
  - `buildPostCompactMessages`: 构建压缩后消息数组
  - `annotateBoundaryWithPreservedSegment`: 注解边界标记

## 功能概述

该文件是 Claude Code 对话压缩系统的核心模块，负责处理当对话上下文接近或超过模型 token 限制时的自动和手动压缩操作。通过生成对话摘要并清理旧消息，释放上下文空间，同时保留重要信息以便会话可以继续。

## 核心内容详解

### 主要常量和配置

```typescript
// 压缩后文件恢复限制
export const POST_COMPACT_MAX_FILES_TO_RESTORE = 5
export const POST_COMPACT_TOKEN_BUDGET = 50_000
export const POST_COMPACT_MAX_TOKENS_PER_FILE = 5_000
export const POST_COMPACT_MAX_TOKENS_PER_SKILL = 5_000
export const POST_COMPACT_SKILLS_TOKEN_BUDGET = 25_000
const MAX_COMPACT_STREAMING_RETRIES = 2
```

### 核心函数

#### `compactConversation`
完整对话压缩，用于处理整个对话历史：
- 执行 PreCompact hooks
- 流式生成对话摘要
- 创建边界标记和摘要消息
- 生成附件（文件、计划、技能等）
- 执行 SessionStart 和 PostCompact hooks
- 支持 Prompt Too Long 重试机制

#### `partialCompactConversation`
部分对话压缩，支持两种方向：
- `'from'`: 从指定索引开始向后摘要，保留前面的消息
- `'up_to'`: 摘要到指定索引为止，保留后面的消息

#### `stripImagesFromMessages`
从消息中移除图片块，避免压缩请求本身超出 token 限制。

#### `createPostCompactFileAttachments`
创建压缩后要恢复的文件附件，基于最近访问的文件状态。

#### `createSkillAttachmentIfNeeded`
创建被调用技能的附件，确保技能指南在压缩后仍然可用。

### 缓存共享优化

使用 `tengu_compact_cache_prefix` 特性开关控制是否复用主对话的 prompt cache：
- 默认启用（3P 环境）
- 使用 `runForkedAgent` 复用缓存前缀
- 失败时回退到常规流式路径

## 设计要点

1. **双路径架构**: 支持完整的 fork-agent 缓存共享路径和流式回退路径
2. **错误恢复**: Prompt Too Long 错误时自动截断旧消息重试
3. **Hook 系统**: 集成 PreCompact、SessionStart、PostCompact hooks 允许扩展
4. **附件恢复**: 智能恢复文件、计划、技能等上下文
5. **边界标记**: 使用 `SystemCompactBoundaryMessage` 标记压缩点
6. **会话记忆**: 支持 KAIROS 特性的会话转录段写入

## 与其他文件的关系

- **prompt.ts**: 提供压缩提示词模板 (`getCompactPrompt`, `getPartialCompactPrompt`)
- **autoCompact.ts**: 调用 `compactConversation` 执行自动压缩
- **sessionMemoryCompact.ts**: 替代传统压缩的会话记忆压缩实现
- **postCompactCleanup.ts**: 压缩后执行清理操作
- **microCompact.ts**: 提供微压缩功能
- **grouping.ts**: 按 API 轮次分组消息

## 注意事项

1. **Token 估算**: 使用 `roughTokenCountEstimationForMessages` 进行快速估算
2. **并发处理**: 附件生成并行执行以提高性能
3. **流式状态**: 压缩期间更新 UI 流式状态（requesting/responding）
4. **错误处理**: 区分用户中止和其他错误，避免不必要的通知
5. **工具禁用**: 压缩期间禁用工具使用（`createCompactCanUseTool`）
