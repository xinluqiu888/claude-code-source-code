# speculation.ts — 推测执行

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/PromptSuggestion/speculation.ts`
- **作用域**: 提示建议的推测执行和文件覆盖层管理
- **主要导出**:
  - `isSpeculationEnabled`: 检查推测执行是否启用
  - `startSpeculation`: 启动推测执行
  - `acceptSpeculation`: 接受推测执行结果
  - `abortSpeculation`: 中止推测执行
  - `handleSpeculationAccept`: 处理推测接受
  - `prepareMessagesForInjection`: 准备消息注入

## 功能概述

当提示建议生成后，使用 forked agent 在后台推测性地执行建议内容。使用文件覆盖层（overlay）隔离文件系统修改，用户接受后合并到主工作区。

## 核心内容详解

### 配置常量

```typescript
const MAX_SPECULATION_TURNS = 20      // 最大轮次
const MAX_SPECULATION_MESSAGES = 100  // 最大消息数

const WRITE_TOOLS = new Set(['Edit', 'Write', 'NotebookEdit'])
const SAFE_READ_ONLY_TOOLS = new Set([
  'Read', 'Glob', 'Grep', 'ToolSearch', 'LSP', 'TaskGet', 'TaskList'
])
```

### 核心函数

#### `startSpeculation(suggestionText, context, setAppState, isPipelined?, cacheSafeParams?)`

启动推测执行：
1. 创建中止控制器（继承自主控制器）
2. 创建临时覆盖层目录
3. 设置推测状态（active）
4. 执行 forked agent 进行推测

**工具权限控制**:
- 写工具：检查权限模式，仅在 `acceptEdits`/`bypassPermissions`/`plan+bypass` 模式下允许
- 安全只读工具：允许，路径重定向到覆盖层
- Bash：仅允许只读命令
- 其他工具：拒绝

**文件路径重写**:
- 写操作：copy-on-write，先复制原文件到覆盖层
- 读操作：优先从覆盖层读取（如果之前写入过）

**边界检测**:
- `edit`: 文件编辑（权限不足）
- `bash`: 非只读 Bash 命令
- `denied_tool`: 被拒绝的工具
- `complete`: 推测完成

#### `acceptSpeculation(state, setAppState, cleanMessageCount)`

接受推测结果：
1. 中止推测 agent
2. 复制覆盖层文件到主工作区（如有清理消息）
3. 清理覆盖层目录
4. 记录遥测
5. 返回推测结果

#### `abortSpeculation(setAppState)`

中止推测执行：
1. 中止推测 agent
2. 清理覆盖层目录
3. 重置推测状态
4. 记录遥测

#### `handleSpeculationAccept(speculationState, speculationSessionTimeSavedMs, setAppState, input, deps)`

处理用户接受推测建议：
1. 清除提示建议状态
2. 准备要注入的消息（清理 thinking、中断消息等）
3. 注入用户消息（即时视觉反馈）
4. 接受推测结果
5. 注入推测消息
6. 更新文件读取状态缓存
7. 生成反馈消息（ant-only）
8. 推广流水线建议（如推测完成）
9. 返回是否需要后续查询

#### `prepareMessagesForInjection(messages)`

准备消息用于注入主对话：
- 过滤 thinking 和 redacted_thinking 块
- 过滤没有成功结果的 tool_use
- 过滤中断消息
- 过滤仅空白内容的消息

#### `generatePipelinedSuggestion(context, suggestionText, speculatedMessages, setAppState, parentAbortController)`

在推测执行期间预生成下一个建议：
- 使用推测结果作为上下文
- 在后台生成下一个建议
- 存储在 `pipelinedSuggestion` 中

### 状态管理

#### 推测状态类型

```typescript
type ActiveSpeculationState = {
  status: 'active'
  id: string
  abort: () => void
  startTime: number
  messagesRef: { current: Message[] }
  writtenPathsRef: { current: Set<string> }
  boundary: CompletionBoundary | null
  suggestionLength: number
  toolUseCount: number
  isPipelined: boolean
  contextRef: { current: REPLHookContext }
  pipelinedSuggestion?: {
    text: string
    promptId: string
    generationRequestId: string
  }
}
```

### 遥测

#### `logSpeculation(id, outcome, startTime, suggestionLength, messages, boundary, extras?)`
记录推测事件：
- outcome: 'accepted' | 'aborted' | 'error'
- 工具执行计数
- 完成状态
- 边界类型/工具/详情

### 反馈消息

#### `createSpeculationFeedbackMessage(messages, boundary, timeSavedMs, sessionTotalMs)`
创建 ant-only 反馈消息：
```
[ANT-ONLY] Speculated N tool uses · X tokens · +Yms saved (Z this session)
```

## 设计要点

1. **文件覆盖层**: 使用临时目录隔离推测的文件系统修改
2. **Copy-on-Write**: 写操作前复制原文件到覆盖层
3. **工具权限**: 严格限制可使用的工具
4. **流水线建议**: 推测期间预生成下一个建议
5. **消息清理**: 注入前清理 thinking、中断等块
6. **边界检测**: 遇到权限边界时优雅停止

## 与其他文件的关系

- **promptSuggestion.ts**: 调用 `startSpeculation`
- **forkedAgent.ts**: 提供 forked agent 执行
- **AppStateStore.ts**: 推测状态管理
- **fileStateCache.ts**: 文件状态缓存合并

## 注意事项

1. **Ant 专用**: `isSpeculationEnabled()` 仅对 ant 用户返回 true
2. **全局配置**: `speculationEnabled` 设置控制功能
3. **覆盖层位置**: `~/.claude/tmp/speculation/<pid>/<id>/`
4. **消息数量限制**: 超过 100 条消息自动中止
5. **轮次限制**: 最大 20 轮
6. **失败开放**: `handleSpeculationAccept` 失败时回退到正常查询流程
