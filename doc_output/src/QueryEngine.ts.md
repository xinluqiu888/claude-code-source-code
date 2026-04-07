# QueryEngine.ts — 查询引擎与会话生命周期管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/QueryEngine.ts`
- **类型**: TypeScript 模块
- **导出内容**: `QueryEngine` 类、`ask()` 函数、`QueryEngineConfig` 类型
- **代码规模**: 约 1295 行
- **依赖关系**: 大量导入，包括 API 客户端、状态管理、工具系统、消息处理等

## 功能概述

QueryEngine 是 Claude Code 的核心组件，负责管理对话的生命周期和会话状态。它将 `ask()` 函数的核心逻辑提取为独立的类，可以同时支持 headless/SDK 路径和未来可能的 REPL 集成。每个 QueryEngine 实例管理一个完整的对话，支持多轮对话、状态持久化、权限跟踪、推测执行等高级功能。

## 核心内容详解

### 1. QueryEngineConfig 类型 (第129-172行)

查询引擎配置接口：

```typescript
export type QueryEngineConfig = {
  cwd: string
  tools: Tools
  commands: Command[]
  mcpClients: MCPServerConnection[]
  agents: AgentDefinition[]
  canUseTool: CanUseToolFn
  getAppState: () => AppState
  setAppState: (f: (prev: AppState) => AppState) => void
  initialMessages?: Message[]
  readFileCache: FileStateCache
  customSystemPrompt?: string
  appendSystemPrompt?: string
  userSpecifiedModel?: string
  fallbackModel?: string
  thinkingConfig?: ThinkingConfig
  maxTurns?: number
  maxBudgetUsd?: number
  taskBudget?: { total: number }
  jsonSchema?: Record<string, unknown>
  verbose?: boolean
  replayUserMessages?: boolean
  handleElicitation?: ToolUseContext['handleElicitation']
  includePartialMessages?: boolean
  setSDKStatus?: (status: SDKStatus) => void
  abortController?: AbortController
  orphanedPermission?: OrphanedPermission
  snipReplay?: (yieldedSystemMsg: Message, store: Message[]) => 
    { messages: Message[]; executed: boolean } | undefined
}
```

**关键配置**:
- `tools/commands/mcpClients/agents`: 可用功能集合
- `getAppState/setAppState`: 状态访问
- `maxTurns/maxBudgetUsd`: 资源限制
- `jsonSchema`: 结构化输出模式
- `snipReplay`: 推测执行/历史裁剪回调

### 2. QueryEngine 类 (第183-1177行)

#### 属性 (第184-197行)

```typescript
private config: QueryEngineConfig
private mutableMessages: Message[]
private abortController: AbortController
private permissionDenials: SDKPermissionDenial[]
private totalUsage: NonNullableUsage
private hasHandledOrphanedPermission = false
private readFileState: FileStateCache
private discoveredSkillNames = new Set<string>()
private loadedNestedMemoryPaths = new Set<string>()
```

#### 构造函数 (第199-207行)

初始化所有属性，使用提供的配置。

#### submitMessage() 方法 (第209-1156行)

提交消息并启动查询循环的核心方法，返回异步生成器。

**主要阶段**:

1. **初始化** (第238-240行):
   - 清除技能发现集合
   - 设置当前目录
   - 检查会话持久化

2. **权限跟踪包装** (第244-271行):
   - 包装 `canUseTool` 以跟踪拒绝
   - 记录到 `permissionDenials` 数组

3. **系统提示构建** (第284-325行):
   - 获取系统提示各部分
   - 处理记忆机制提示
   - 组装完整系统提示

4. **ProcessUserInputContext 创建** (第335-394行):
   - 创建消息处理上下文
   - 设置各种状态更新器
   - 配置技能发现跟踪

5. **孤儿权限处理** (第397-408行):
   - 处理遗留的权限请求
   - 只处理一次（生命周期限制）

6. **用户输入处理** (第410-428行):
   - 调用 `processUserInput()`
   - 获取处理后的消息和元数据

7. **转录持久化** (第450-463行):
   - 在进入查询循环前写入转录
   - 支持 fire-and-forget 和等待模式

8. **消息确认** (第466-474行):
   - 过滤可确认的消息
   - 用于 SDK 消息重放

9. **主查询循环** (第675-1049行):
   - 调用 `query()` 函数
   - 处理各种消息类型（assistant, user, progress, system 等）
   - 跟踪 token 使用
   - 处理各种边界条件（预算、回合数、结构化输出重试）

10. **结果处理** (第1051-1156行):
    - 检查结果是否成功
    - 生成成功或错误结果
    - 刷新转录写入

#### interrupt() 方法 (第1158-1160行)

中断查询：
```typescript
interrupt(): void {
  this.abortController.abort()
}
```

#### 获取方法 (第1162-1176行)

```typescript
getMessages(): readonly Message[]
getReadFileState(): FileStateCache
getSessionId(): string
setModel(model: string): void
```

### 3. ask() 函数 (第1186-1295行)

单次提示的便利包装器，使用 QueryEngine。

**设计说明**:
- 适用于非交互式使用
- 不会询问用户权限或进一步输入
- 完整的异步生成器，产生 SDKMessage

**参数**: 与 QueryEngineConfig 大部分重叠，增加了：
- `prompt`: 提示内容
- `promptUuid`: 提示 UUID
- `isMeta`: 是否元消息
- `mutableMessages`: 可变消息数组
- `getReadFileCache/setReadFileCache`: 文件缓存访问器

**实现** (第1249-1294行):
1. 创建 QueryEngine 实例
2. 配置 snipReplay（如果启用 HISTORY_SNIP）
3. 调用 `submitMessage()`
4. finally 中保存文件缓存状态

## 设计要点

1. **状态隔离**: 每个 QueryEngine 实例管理独立的对话状态
2. **异步生成器**: 使用 `AsyncGenerator` 流式返回结果
3. **资源跟踪**: 跟踪 token 使用、成本、权限拒绝
4. **推测执行**: 支持历史裁剪以管理长会话内存
5. **多轮对话**: 支持 `submitMessage` 多次调用的对话连续性
6. **错误边界**: 处理各种错误条件（预算、回合数、API 错误）

## 与其他文件的关系

- **query.ts**: 核心查询循环实现
- **processUserInput.ts**: 用户输入处理
- **AppStateStore.ts**: 状态管理
- **utils/sessionStorage.ts**: 转录持久化
- **cost-tracker.ts**: 成本跟踪

## 使用场景

```typescript
// SDK 用法
const engine = new QueryEngine({
  cwd: '/project',
  tools: myTools,
  commands: myCommands,
  mcpClients: [],
  agents: [],
  canUseTool: checkPermission,
  getAppState: () => appState,
  setAppState: updateState,
  readFileCache: createFileStateCache(),
  maxTurns: 10,
  maxBudgetUsd: 1.0,
})

for await (const message of engine.submitMessage('Hello')) {
  console.log(message)
}

// 或者使用便捷函数
for await (const message of ask({
  prompt: 'Hello',
  cwd: '/project',
  tools: myTools,
  // ... 其他选项
})) {
  console.log(message)
}
```

## 注意事项

1. **会话持久化**: `--bare` 模式下转录写入是 fire-and-forget
2. **token 使用**: 使用 `accumulateUsage` 和 `updateUsage` 跟踪
3. **权限拒绝**: 所有拒绝记录到 `permissionDenials` 供 SDK 报告
4. **模型切换**: 支持通过 slash 命令在对话中切换模型
5. **内存管理**: `snipReplay` 支持长会话的内存管理
