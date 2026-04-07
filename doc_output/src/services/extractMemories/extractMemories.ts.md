# extractMemories.ts — 会话记忆提取

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/extractMemories/extractMemories.ts`
- **作用域**: 从会话转录提取持久化记忆
- **主要导出**:
  - `initExtractMemories`: 初始化记忆提取系统
  - `executeExtractMemories`: 执行记忆提取
  - `drainPendingExtraction`: 等待所有提取完成
  - `createAutoMemCanUseTool`: 创建自动记忆工具权限控制

## 功能概述

在每个查询循环结束时（当模型生成最终响应且没有工具调用时）运行，从当前会话转录中提取持久化记忆并写入自动记忆目录。使用 forked agent 模式，与主对话共享 prompt cache。

## 核心内容详解

### 初始化与状态管理

```typescript
export function initExtractMemories(): void
```

创建闭包作用域的状态：
- `inFlightExtractions`: 正在执行的提取 Promise 集合
- `lastMemoryMessageUuid`: 最后处理消息的 UUID 游标
- `hasLoggedGateFailure`: 门控失败日志标记
- `inProgress`: 执行中标记
- `turnsSinceLastExtraction`: 自上次提取以来的轮次计数
- `pendingContext`: 暂存的后续执行上下文

### 核心提取逻辑

#### `runExtraction({ context, appendSystemMessage, isTrailingRun })`

1. **消息计数**: 计算自上次提取后的新消息数
2. **直接写入检查**: 如果主代理已写入记忆文件，跳过提取
3. **特性检查**: 检查 `TEAMMEM` 和团队记忆启用状态
4. **节流连拍**: 每 N 轮执行一次（`tengu_bramble_lintel`，默认 1）
5. **预扫描记忆**: 扫描现有记忆文件生成清单
6. **构建提示词**: 根据模式选择 `buildExtractAutoOnlyPrompt` 或 `buildExtractCombinedPrompt`
7. **执行 forked agent**: 运行提取代理（最多 5 轮）
8. **游标推进**: 成功后推进游标
9. **结果处理**: 提取写入路径，记录遥测，生成系统消息

### 工具权限控制

#### `createAutoMemCanUseTool(memoryDir)`

返回 `CanUseToolFn` 控制提取代理的工具使用：

| 工具 | 权限 |
|------|------|
| REPL | 允许（VM 会重新检查底层工具） |
| Read/Grep/Glob | 允许（只读） |
| Bash | 仅允许只读命令（ls, find, cat 等） |
| Edit/Write | 仅允许写入记忆目录 |

### 公共 API

#### `executeExtractMemories(context, appendSystemMessage?)`
查询循环结束时调用：
- 仅主代理执行（跳过子代理）
- 检查 `tengu_passport_quail` 特性标志
- 检查自动记忆启用状态
- 检查远程模式
- 支持并发合并（暂存后续执行）

#### `drainPendingExtraction(timeoutMs?)`
等待所有提取完成（软超时）：
- print.ts 在响应刷新后调用
- 在 gracefulShutdownSync 前完成 forked agent

### 辅助函数

#### `isModelVisibleMessage(message)`
判断消息是否对模型可见（排除 progress、system、attachment）

#### `hasMemoryWritesSince(messages, sinceUuid)`
检查自游标后是否有助手消息包含对自动记忆路径的 Write/Edit 工具调用

#### `extractWrittenPaths(agentMessages)`
从代理消息中提取写入的文件路径

## 设计要点

1. **游标机制**: 基于 UUID 的游标确保只处理新消息
2. **互斥执行**: 主代理直接写入时跳过 forked 提取
3. **并发控制**: in-progress 时暂存后续执行
4. **工具限制**: 严格限制可使用的工具和路径
5. **节流连拍**: 可配置每 N 轮执行一次
6. **最佳努力**: 提取失败不阻塞主流程

## 与其他文件的关系

- **prompts.ts**: 提供提取提示词构建
- **forkedAgent.ts**: 提供 forked agent 执行
- **memoryScan.ts**: 扫描现有记忆文件
- **teamMemPaths.ts**: 团队记忆路径（TEAMMEM 特性）

## 注意事项

1. **Feature Flags**:
   - `tengu_passport_quail`: 主开关
   - `tengu_bramble_lintel`: 节流连拍间隔
   - `TEAMMEM`: 团队记忆支持

2. **提取限制**:
   - 最大 5 轮
   - 跳过 transcript 记录
   - 最佳努力，失败不通知用户

3. **路径安全**: 仅允许写入 `~/.claude/projects/<path>/memory/`
