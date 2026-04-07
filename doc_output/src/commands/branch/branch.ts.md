# branch.ts — 会话分支创建逻辑

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/branch/branch.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 296 行 |
| 主要职责 | 实现会话分支创建功能，允许从当前对话点创建分支 |

## 功能概述

该文件实现了 `/branch` 命令的核心逻辑，用于在当前对话的某个点创建一个分支会话。分支功能类似于 Git 的分支，允许用户在保持原始会话的同时，从特定点开始一个新的会话线程。分支会复制原始会话的消息历史，但使用新的会话 ID，并建立可追溯的关联关系。

## 核心内容详解

### 导入依赖

```typescript
import { randomUUID, type UUID } from 'crypto'
import { mkdir, readFile, writeFile } from 'fs/promises'
import { getOriginalCwd, getSessionId } from '../../bootstrap/state.js'
import type { LocalJSXCommandContext } from '../../commands.js'
import { logEvent } from '../../services/analytics/index.js'
import type { LocalJSXCommandOnDone } from '../../types/command.js'
import type { ContentReplacementEntry, Entry, LogOption, SerializedMessage, TranscriptMessage } from '../../types/logs.js'
import { parseJSONL } from '../../utils/json.js'
import { getProjectDir, getTranscriptPath, getTranscriptPathForSession, isTranscriptMessage, saveCustomTitle, searchSessionsByCustomTitle } from '../../utils/sessionStorage.js'
import { jsonStringify } from '../../utils/slowOperations.js'
import { escapeRegExp } from '../../utils/stringUtils.js'
```

### 核心函数

**deriveFirstPrompt**
- 从第一条用户消息派生分支标题
- 处理多行内容，将换行符替换为空格
- 截取前 100 个字符作为标题
- 回退文本：'Branched conversation'

**createFork**
- 核心分支创建函数
- 流程：
  1. 生成新的会话 UUID
  2. 读取当前会话的 transcript 文件
  3. 解析所有消息条目
  4. 过滤出主对话消息（排除 sidechain）
  5. 复制 content-replacement 记录
  6. 构建新的分支消息（更新 sessionId、添加 forkedFrom 追踪）
  7. 写入分支会话文件
  8. 返回分支信息

**getUniqueForkName**
- 生成唯一的分支名称
- 检查名称冲突（如 "Title (Branch)" 已存在则尝试 "Title (Branch 2)")  
- 支持自定义标题或使用派生的第一提示作为基础

**call**
- 命令入口函数
- 接收可选的自定义标题参数
- 调用 `createFork` 创建分支
- 保存自定义标题
- 记录分析事件
- 构建 LogOption 用于恢复分支
- 调用 `context.resume` 切换到新分支
- 返回成功消息和恢复原始会话的提示

### TranscriptEntry 类型

```typescript
type TranscriptEntry = TranscriptMessage & {
  forkedFrom?: {
    sessionId: string
    messageUuid: UUID
  }
}
```

用于记录每个分支消息的来源信息，实现可追溯性。

## 设计要点

1. **可追溯性**：通过 `forkedFrom` 字段记录每个消息的来源，建立完整的分支谱系
2. **内容保留**：保留 content-replacement 记录，确保分支会话正确处理缓存替换
3. **唯一命名**：自动处理标题冲突，生成唯一的分支名称
4. **无缝切换**：通过 `context.resume` 立即切换到新分支
5. **分析追踪**：记录分支创建事件，包含消息数量和自定义标题使用情况

## 与其他文件的关系

- **sessionStorage.ts**: 提供会话存储和标题管理功能
- **json.ts**: 提供 JSONL 解析功能
- **analytics/index.ts**: 提供事件记录功能
- **types/logs.ts**: 定义日志相关类型
- **index.ts**: 命令注册配置

## 注意事项

1. **Session ID 更新**：分支使用全新的 session ID，原始会话 ID 作为父会话记录
2. **文件权限**：写入分支文件时设置 `mode: 0o600`（仅所有者可读写）
3. **错误处理**：在文件读取失败时抛出 'No conversation to branch' 错误
4. **Symlink 更新**：对于保留的后台任务，会重新指向新的会话路径
