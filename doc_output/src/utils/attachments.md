# attachments.ts — 消息附件生成系统

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/attachments.ts`
- **主要功能**: 为消息生成各种附件（文件、待办、任务、诊断等）
- **关键依赖**: 大量，包括 Tool 系统、状态管理、MCP 等

## 功能概述

该模块负责为 Claude Code 的消息生成各种附件：
1. 用户输入相关附件（@提及文件、Agent 提及）
2. 线程安全附件（所有线程可用）
3. 主线程专用附件（IDE 选择、诊断等）
4. 技能发现、计划模式、自动模式附件
5. 队友/团队相关附件

## 核心内容详解

### 附件类型定义

```typescript
export type Attachment =
  | FileAttachment                    // @提及文件
  | CompactFileReferenceAttachment   // 压缩文件引用
  | PDFReferenceAttachment           // PDF 引用
  | AlreadyReadFileAttachment        // 已读文件
  | { type: 'edited_text_file' }     // 编辑的文本文件
  | { type: 'directory' }            // 目录内容
  | { type: 'selected_lines_in_ide' }// IDE 选中行
  | { type: 'todo_reminder' }        // 待办提醒
  | { type: 'task_reminder' }        // 任务提醒
  | { type: 'relevant_memories' }    // 相关记忆
  | { type: 'skill_discovery' }      // 技能发现
  | { type: 'plan_mode' }            // 计划模式
  | { type: 'auto_mode' }            // 自动模式
  | { type: 'teammate_mailbox' }     // 队友邮箱
  | { type: 'diagnostics' }          // 诊断信息
  | // ... 更多类型
```

### 配置常量

```typescript
export const TODO_REMINDER_CONFIG = {
  TURNS_SINCE_WRITE: 10,
  TURNS_BETWEEN_REMINDERS: 10,
}

export const PLAN_MODE_ATTACHMENT_CONFIG = {
  TURNS_BETWEEN_ATTACHMENTS: 5,
  FULL_REMINDER_EVERY_N_ATTACHMENTS: 5,
}

export const MAX_MEMORY_LINES = 200
export const MAX_MEMORY_BYTES = 4096
```

### 核心函数

#### getAttachments

```typescript
export async function getAttachments(
  input: string | null,
  toolUseContext: ToolUseContext,
  ideSelection: IDESelection | null,
  queuedCommands: QueuedCommand[],
  messages?: Message[],
  querySource?: QuerySource,
  options?: { skipSkillDiscovery?: boolean }
): Promise<Attachment[]>
```

主要附件生成函数，按以下顺序处理：

1. **用户输入附件**（并行处理）：
   - @提及文件处理
   - MCP 资源附件
   - Agent 提及处理
   - 技能发现

2. **线程安全附件**（所有线程可用）：
   - 排队命令
   - 日期变更
   - ultrathink 努力级别
   - 延迟工具变更
   - Agent 列表变更
   - MCP 指令变更
   - 伴侣介绍
   - 变更文件
   - 嵌套记忆
   - 动态技能
   - 技能列表
   - 计划模式
   - 自动模式
   - 待办/任务提醒
   - 队友邮箱/团队上下文
   - 关键系统提醒

3. **主线程附件**（仅主线程）：
   - IDE 选择
   - IDE 打开文件
   - 输出样式
   - 诊断信息
   - LSP 诊断
   - 统一任务
   - 异步 hook 响应
   - Token 使用情况
   - 预算 USD
   - 输出 token 使用
   - 验证计划提醒

### 辅助函数

#### maybe

```typescript
async function maybe<A>(label: string, f: () => Promise<A[]>): Promise<A[]>
```

包装附件生成函数，提供：
- 错误处理（返回空数组）
- 性能日志（5% 采样率）
- 错误日志记录

#### getQueuedCommandAttachments

```typescript
export async function getQueuedCommandAttachments(
  queuedCommands: QueuedCommand[]
): Promise<Attachment[]>
```

处理排队命令附件，支持图像粘贴内容。

## 设计要点

1. **分层处理**: 按依赖关系分三层处理附件
2. **超时保护**: 1秒超时防止附件生成阻塞
3. **并行处理**: 同层附件并行生成
4. **错误隔离**: `maybe` 包装确保单个附件失败不影响其他
5. **性能监控**: 5% 采样率记录附件生成性能
6. **功能门控**: 大量特性使用 `feature()` 检查

## 与其他文件的关系

| 文件/模块 | 关系 |
|----------|------|
| `Tool.js` | 工具匹配和上下文类型 |
| `FileReadTool` | 文件读取和图像处理 |
| `bootstrap/state.js` | 状态管理 |
| `todo/types.js` | 待办类型 |
| `tasks.js` | 任务管理 |
| `claudemd.js` | 记忆文件 |
| `ide.js` | IDE 集成 |
| `permissions/filesystem.js` | 权限检查 |
| `teammate*.js` | 队友/团队功能 |
| `skillSearch/*` | 技能搜索（条件 require） |
| `mcpInstructionsDelta.js` | MCP 指令变更 |

## 注意事项

1. **附件顺序**: 用户输入附件先处理以填充依赖
2. **超时**: 整体1秒超时，单个附件可能未完成
3. **过滤**: `isEnvTruthy(CLAUDE_CODE_DISABLE_ATTACHMENTS)` 禁用大部分附件
4. **主线程限制**: 部分附件仅在主线程生成
5. **技能发现跳过**: `skipSkillDiscovery` 选项避免 SKILL.md 触发搜索
