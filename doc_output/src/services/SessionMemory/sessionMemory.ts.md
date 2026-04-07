# sessionMemory.ts — 会话记忆管理

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/SessionMemory/sessionMemory.ts`
- **作用域**: 会话记忆的自动提取和更新
- **主要导出**:
  - `initSessionMemory`: 初始化会话记忆系统
  - `shouldExtractMemory`: 检查是否应该提取记忆
  - `manuallyExtractSessionMemory`: 手动触发记忆提取
  - `createMemoryFileCanUseTool`: 创建记忆文件工具权限控制

## 功能概述

自动维护一个关于当前对话的 Markdown 笔记文件。使用 forked 子代理定期在后台运行，提取关键信息而不中断主对话流程。会话记忆可用于压缩时替代传统摘要。

## 核心内容详解

### 配置和特性门控

```typescript
// 特性门控（缓存值，非阻塞）
function isSessionMemoryGateEnabled(): boolean {
  return getFeatureValue_CACHED_MAY_BE_STALE('tengu_session_memory', false)
}

// 远程配置（缓存值，非阻塞）
function getSessionMemoryRemoteConfig(): Partial<SessionMemoryConfig> {
  return getDynamicConfig_CACHED_MAY_BE_STALE<Partial<SessionMemoryConfig>>(
    'tengu_sm_config',
    {},
  )
}
```

### 主要函数

#### `initSessionMemory()`
初始化会话记忆系统：
1. 检查远程模式
2. 检查自动压缩启用状态
3. 注册 post-sampling hook

门控检查和配置加载在 hook 运行时延迟执行。

#### `shouldExtractMemory(messages)`
检查是否应该提取记忆：

**初始化检查**: 
- 检查是否达到 `minimumMessageTokensToInit`（默认 10,000）
- 首次达到后标记为已初始化

**更新阈值检查**:
- Token 增长检查：`minimumTokensBetweenUpdate`（默认 5,000）
- 工具调用检查：`toolCallsBetweenUpdates`（默认 3）

**提取触发条件**:
1. Token 和工具调用阈值都达到，或
2. Token 阈值达到且最后一轮助手消息没有工具调用

**重要**: Token 阈值始终是必需的，即使工具调用阈值达到，也必须满足 Token 阈值才能提取。

#### `extractSessionMemory(context)`
核心提取逻辑（sequential 包装）：
1. 仅主 REPL 线程执行
2. 检查特性门控
3. 初始化配置
4. 检查提取条件
5. 设置会话记忆文件
6. 构建更新提示词
7. 运行 forked agent 提取记忆
8. 记录遥测
9. 更新 `lastSummarizedMessageId`

#### `setupSessionMemoryFile(toolUseContext)`
设置会话记忆文件：
1. 创建目录（权限 0o700）
2. 如不存在则创建文件并写入模板（权限 0o600）
3. 读取当前记忆内容
4. 返回路径和内容

#### `manuallyExtractSessionMemory(messages, toolUseContext)`
手动触发提取（`/summary` 命令使用）：
- 绕过阈值检查
- 使用相同的提取逻辑

### 工具权限控制

#### `createMemoryFileCanUseTool(memoryPath)`
创建仅允许编辑指定记忆文件的权限函数：
- 仅允许 `FILE_EDIT_TOOL_NAME`
- 仅允许指定的 `memoryPath`

### 配置初始化

#### `initSessionMemoryConfigIfNeeded()`
从远程配置初始化（memoized，仅执行一次）：
```typescript
const config: SessionMemoryConfig = {
  minimumMessageTokensToInit: 10_000,    // 初始化阈值
  minimumTokensBetweenUpdate: 5_000,     // 更新间隔阈值
  toolCallsBetweenUpdates: 3,            // 工具调用间隔
}
```

### 辅助函数

#### `countToolCallsSince(messages, sinceUuid)`
计算自指定 UUID 以来的工具调用数

#### `updateLastSummarizedMessageIdIfSafe(messages)`
安全地更新最后摘要消息 ID（仅当最后一轮没有工具调用时）

## 设计要点

1. **延迟门控**: 特性检查在 hook 运行时执行，非阻塞
2. **缓存配置**: 使用 GrowthBook 缓存值，避免初始化阻塞
3. **Sequential 包装**: 防止并发提取
4. **Forked Agent**: 复用主对话的 prompt cache
5. **阈值控制**: 双重阈值（token + 工具调用）防止过度提取
6. **安全更新**: 仅在无工具调用时更新 `lastSummarizedMessageId`

## 与其他文件的关系

- **prompts.ts**: 提供提示词构建和模板加载
- **sessionMemoryUtils.ts**: 提供配置管理和状态追踪
- **forkedAgent.ts**: 提供 forked agent 执行
- **postSamplingHooks.ts**: 注册 post-sampling hook

## 注意事项

1. **Feature Flags**:
   - `tengu_session_memory`: 主开关
   - `tengu_sm_config`: 配置

2. **文件位置**: `~/.claude/projects/<path>/.claude/SESSION_MEMORY.md`
3. **文件权限**: 目录 0o700，文件 0o600
4. **自动压缩依赖**: 会话记忆启用依赖于自动压缩启用
5. **主线程专用**: 仅 `repl_main_thread` 执行
6. **远程模式禁用**: 远程模式下禁用
