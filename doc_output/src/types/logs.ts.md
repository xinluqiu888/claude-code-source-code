# logs.ts — 会话日志与消息条目类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/types/logs.ts`
- **类型**: TypeScript 模块
- **导出内容**: 日志选项、序列化消息、各种条目类型、日志排序函数
- **依赖关系**:
  - 导入: `crypto`, `fileHistory.js`, `toolResultStorage.js`, `ids.js`, `message.js`, `messageQueueTypes.js`

## 功能概述

本文件定义了 Claude Code 会话日志系统的完整类型体系，包括持久化的日志选项、序列化消息格式、各种类型的条目（摘要、标题、标签、PR 链接等），以及上下文压缩相关类型。它是会话存储、恢复和转录的基础。

## 核心内容详解

### 1. SerializedMessage (第8-17行)

序列化消息类型，在 Message 基础上添加元数据：

```typescript
export type SerializedMessage = Message & {
  cwd: string                    // 当前工作目录
  userType: string               // 用户类型
  entrypoint?: string            // 入口点（区分 cli/sdk-ts/sdk-py 等）
  sessionId: string              // 会话ID
  timestamp: string              // ISO时间戳
  version: string                // 版本
  gitBranch?: string             // Git分支
  slug?: string                  // 会话标识（用于恢复）
}
```

### 2. LogOption (第19-53行)

日志选项的完整定义，用于 /resume 命令：

**基础字段**:
- `date`: 日期字符串
- `messages`: 序列化消息数组
- `fullPath`: 完整路径（可选）
- `value`: 数值
- `created/modified`: 创建和修改时间

**内容统计**:
- `firstPrompt`: 第一个提示
- `messageCount`: 消息数量
- `fileSize`: 文件大小（字节）

**会话元数据**:
- `isSidechain`: 是否是侧链
- `isLite`: 轻量日志（消息未加载）
- `sessionId`: 会话ID（轻量日志）
- `teamName`: 团队名称
- `agentName/agentColor`: 代理名称和颜色
- `agentSetting`: 代理定义
- `isTeammate`: 是否队友会话

**恢复控制**:
- `leafUuid`: 必须包含的 UUID
- `summary`: 会话摘要
- `customTitle`: 自定义标题
- `tag`: 可搜索标签
- `fileHistorySnapshots`: 文件历史快照
- `attributionSnapshots`: 归属快照
- `contextCollapseCommits/contextCollapseSnapshot`: 上下文压缩状态

**Git 和 PR**:
- `gitBranch`: Git分支
- `projectPath`: 项目目录
- `prNumber/prUrl/prRepository`: PR信息

**模式**:
- `mode`: 会话模式（coordinator/normal）
- `worktreeSession`: 工作树会话状态

**内容**:
- `contentReplacements`: 内容替换记录

### 3. 各种消息条目类型

#### SummaryMessage (第55-59行)
```typescript
export type SummaryMessage = {
  type: 'summary'
  leafUuid: UUID
  summary: string
}
```

#### CustomTitleMessage (第61-65行)
用户设置的自定义标题。

#### AiTitleMessage (第75-79行)
AI 生成的会话标题，与 CustomTitleMessage 区分以便：
- 用户重命名始终优先于 AI 标题
- AI 标题不重复追加
- VS Code 的 onlyIfNoCustomTitle 检查

#### LastPromptMessage (第81-85行)
最后提示内容。

#### TaskSummaryMessage (第93-98行)
周期性 fork 生成的任务摘要，用于 `claude ps` 显示更有用的信息。

#### TagMessage (第100-104行)
可搜索标签。

#### AgentNameMessage & AgentColorMessage (第106-116行)
代理名称和颜色。

#### AgentSettingMessage (第118-122行)
代理设置定义。

#### PRLinkMessage (第128-135行)
链接会话到 GitHub PR：
```typescript
export type PRLinkMessage = {
  type: 'pr-link'
  sessionId: UUID
  prNumber: number
  prUrl: string
  prRepository: string  // "owner/repo" 格式
  timestamp: string
}
```

#### ModeEntry (第137-141行)
会话模式记录（coordinator/normal）。

### 4. PersistedWorktreeSession (第149-159行)

持久化的工作树会话状态：

```typescript
export type PersistedWorktreeSession = {
  originalCwd: string
  worktreePath: string
  worktreeName: string
  worktreeBranch?: string
  originalBranch?: string
  originalHeadCommit?: string
  sessionId: string
  tmuxSessionName?: string
  hookBased?: boolean
}
```

### 5. WorktreeStateEntry (第167-171行)

工作树状态条目：
```typescript
export type WorktreeStateEntry = {
  type: 'worktree-state'
  sessionId: UUID
  worktreeSession: PersistedWorktreeSession | null  // null = 已退出
}
```

### 6. ContentReplacementEntry (第181-186行)

内容块替换记录：
```typescript
export type ContentReplacementEntry = {
  type: 'content-replacement'
  sessionId: UUID
  agentId?: AgentId  // 设置表示子代理侧链
  replacements: ContentReplacementRecord[]
}
```

### 7. FileHistorySnapshotMessage (第188-193行)

文件历史快照：
```typescript
export type FileHistorySnapshotMessage = {
  type: 'file-history-snapshot'
  messageId: UUID
  snapshot: FileHistorySnapshot
  isSnapshotUpdate: boolean
}
```

### 8. AttributionSnapshotMessage (第208-219行)

归属状态快照，跟踪 Claude 的字符贡献：

```typescript
export type AttributionSnapshotMessage = {
  type: 'attribution-snapshot'
  messageId: UUID
  surface: string  // 客户端表面（cli, ide, web, api）
  fileStates: Record<string, FileAttributionState>
  promptCount?: number
  promptCountAtLastCommit?: number
  permissionPromptCount?: number
  permissionPromptCountAtLastCommit?: number
  escapeCount?: number
  escapeCountAtLastCommit?: number
}
```

**FileAttributionState** (第198-202行):
```typescript
export type FileAttributionState = {
  contentHash: string    // 内容 SHA-256
  claudeContribution: number  // Claude 写入字符数
  mtime: number          // 修改时间
}
```

### 9. TranscriptMessage (第221-231行)

转录消息，继承 SerializedMessage：

```typescript
export type TranscriptMessage = SerializedMessage & {
  parentUuid: UUID | null
  logicalParentUuid?: UUID | null  // 会话断开时保留逻辑父级
  isSidechain: boolean
  gitBranch?: string
  agentId?: string
  teamName?: string
  agentName?: string
  agentColor?: string
  promptId?: string  // 关联 OTel prompt.id
}
```

### 10. SpeculationAcceptMessage (第233-237行)

推测执行接受消息：
```typescript
export type SpeculationAcceptMessage = {
  type: 'speculation-accept'
  timestamp: string
  timeSavedMs: number
}
```

### 11. 上下文压缩相关类型

#### ContextCollapseCommitEntry (第255-269行)

上下文压缩提交记录：
```typescript
export type ContextCollapseCommitEntry = {
  type: 'marble-origami-commit'  // 混淆的名称
  sessionId: UUID
  collapseId: string             // 16位压缩ID
  summaryUuid: string            // 摘要占位符 UUID
  summaryContent: string         // <collapsed> 文本
  summary: string                // 纯摘要文本
  firstArchivedUuid: string      // 跨度起始
  lastArchivedUuid: string       // 跨度结束
}
```

**设计说明**:
- 类型名称混淆以匹配特性门控名称
- 不持久化已归档消息本身（已存在转录中）
- 只持久化足够重建的信息

#### ContextCollapseSnapshotEntry (第282-295行)

上下文压缩快照条目：
```typescript
export type ContextCollapseSnapshotEntry = {
  type: 'marble-origami-snapshot'
  sessionId: UUID
  staged: Array<{
    startUuid: string
    endUuid: string
    summary: string
    risk: number
    stagedAt: number
  }>
  armed: boolean
  lastSpawnTokens: number
}
```

**特点**:
- 与提交不同，快照是 last-wins（最后获胜）
- 仅应用最近的快照条目
- 在 ctx-agent spawn 解析后写入

### 12. Entry 联合类型 (第297-317行)

所有可能的条目类型联合：
```typescript
export type Entry =
  | TranscriptMessage
  | SummaryMessage
  | CustomTitleMessage
  | AiTitleMessage
  | LastPromptMessage
  | TaskSummaryMessage
  | TagMessage
  | AgentNameMessage
  | AgentColorMessage
  | AgentSettingMessage
  | PRLinkMessage
  | FileHistorySnapshotMessage
  | AttributionSnapshotMessage
  | QueueOperationMessage
  | SpeculationAcceptMessage
  | ModeEntry
  | WorktreeStateEntry
  | ContentReplacementEntry
  | ContextCollapseCommitEntry
  | ContextCollapseSnapshotEntry
```

### 13. sortLogs() 函数 (第319-330行)

日志排序函数：

```typescript
export function sortLogs(logs: LogOption[]): LogOption[] {
  return logs.sort((a, b) => {
    // 优先按修改时间（新到旧）
    const modifiedDiff = b.modified.getTime() - a.modified.getTime()
    if (modifiedDiff !== 0) return modifiedDiff
    // 其次按创建时间
    return b.created.getTime() - a.created.getTime()
  })
}
```

## 设计要点

1. **分离关注点**: 序列化消息添加元数据，与运行时消息分离
2. **丰富的元数据**: LogOption 包含完整的会话元数据用于恢复
3. **类型区分**: AI 标题与用户自定义标题分开处理
4. **上下文压缩**: 专门类型支持高级压缩功能
5. **归属跟踪**: 详细的字符级贡献跟踪
6. **工作树支持**: 完整的工作树会话状态持久化

## 与其他文件的关系

- **utils/sessionStorage.ts**: 使用这些类型进行转录存储
- **commands/resume/**: 使用 LogOption 恢复会话
- **types/message.ts**: SerializedMessage 继承 Message

## 注意事项

1. **混淆类型名**: ContextCollapse 类型使用混淆名称以避免特性门控泄漏
2. **轻量日志**: isLite 标志控制是否加载完整消息
3. **条目联合**: Entry 类型用于通用转录条目处理
4. **last-wins**: ContextCollapseSnapshotEntry 只使用最新的
