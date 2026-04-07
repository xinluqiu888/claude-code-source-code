# sessionMemoryCompact.ts — 会话记忆压缩

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/compact/sessionMemoryCompact.ts`
- **作用域**: 基于会话记忆文件的轻量级压缩
- **主要导出**:
  - `trySessionMemoryCompaction`: 尝试使用会话记忆进行压缩
  - `calculateMessagesToKeepIndex`: 计算要保留的消息起始索引
  - `adjustIndexToPreserveAPIInvariants`: 调整索引以保持 API 不变性
  - `shouldUseSessionMemoryCompaction`: 检查是否应使用会话记忆压缩
  - `SessionMemoryCompactConfig`: 配置类型

## 功能概述

实现实验性的会话记忆压缩功能。利用已提取的会话记忆文件作为摘要，避免调用压缩 API，显著降低成本和延迟。

## 核心内容详解

### 配置

```typescript
export type SessionMemoryCompactConfig = {
  minTokens: number           // 压缩后保留的最小 tokens
  minTextBlockMessages: number // 保留的最少文本块消息数
  maxTokens: number          // 压缩后保留的最大 tokens（硬上限）
}

const DEFAULT_SM_COMPACT_CONFIG: SessionMemoryCompactConfig = {
  minTokens: 10_000,
  minTextBlockMessages: 5,
  maxTokens: 40_000,
}
```

### 核心函数

#### `trySessionMemoryCompaction(messages, agentId?, autoCompactThreshold?)`
主入口函数：

1. **特性检查**: 确认 `tengu_session_memory` 和 `tengu_sm_compact` 标志
2. **配置初始化**: 从 GrowthBook 获取远程配置（仅一次）
3. **等待提取完成**: 等待会话记忆提取完成
4. **会话记忆检查**: 
   - 检查会话记忆文件是否存在
   - 检查是否为空模板
5. **边界计算**: 
   - 找到 `lastSummarizedMessageId`
   - 计算要保留的消息范围
   - 确保工具对不分割
6. **结果构建**: 创建 `CompactionResult`
7. **阈值检查**: 确保压缩后不超过自动压缩阈值

#### `calculateMessagesToKeepIndex(messages, lastSummarizedIndex)`
计算保留消息的起始索引：

1. 从 `lastSummarizedIndex + 1` 开始
2. 向后扩展直到满足最小 tokens 和最小消息数
3. 不超过最大 tokens 限制
4. 不低于最近边界（避免 REPL 剪枝问题）
5. 调整以保持工具对和 thinking 块

#### `adjustIndexToPreserveAPIInvariants(messages, startIndex)`
调整索引以保持 API 不变性：

1. **工具对处理**: 
   - 收集保留范围内所有 `tool_result` ID
   - 向后查找匹配的 `tool_use`
   - 确保所有工具对完整

2. **Thinking 块处理**:
   - 收集保留范围内所有 `message.id`
   - 向后查找相同 ID 的消息（可能包含 thinking 块）
   - 确保 streaming 消息正确合并

#### `hasTextBlocks(message)`
检查消息是否包含文本块（用于最小消息数计算）

## 设计要点

1. **零 API 调用**: 利用已存在的会话记忆文件，无需额外 API 调用
2. **智能边界**: 保留最近消息的同时确保工具对不分割
3. **配置动态化**: 从 GrowthBook 获取配置，支持动态调整
4. **双场景支持**: 
   - 正常情况：有 `lastSummarizedMessageId`
   - 恢复会话：无 ID 但会话记忆有内容

## 与其他文件的关系

- **sessionMemoryUtils.ts**: 提供会话记忆内容获取
- **prompts.ts**: 提供会话记忆模板检查
- **compact.ts**: 复用 `CompactionResult` 构建逻辑

## 注意事项

1. **环境变量**:
   - `ENABLE_CLAUDE_CODE_SM_COMPACT`: 强制启用
   - `DISABLE_CLAUDE_CODE_SM_COMPACT`: 强制禁用

2. **Feature Flags**:
   - `tengu_session_memory`
   - `tengu_sm_compact`
   - `tengu_sm_compact_config`

3. **失败回退**: 任何错误都回退到传统压缩
