# Tool.ts — 工具类型系统与构建工厂

> **一句话总结**：定义工具（Tool）的完整类型契约、执行上下文、权限模型，以及 `buildTool` 工厂函数。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/Tool.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 792 |
| 主要职责 | 声明整个工具系统的类型体系，包括 Tool 接口、ToolUseContext 上下文、权限相关类型，以及通过 `buildTool` 为工具提供默认实现。 |

---

## 功能概述

`Tool.ts` 是 Claude Code 工具系统的核心类型文件。它定义了所有工具必须实现的 `Tool` 接口，包括执行（`call`）、描述（`description`）、权限检查（`checkPermissions`）、UI 渲染（多个 `render*` 方法）等约40个方法/属性。

该文件还定义了工具执行时使用的 `ToolUseContext` 上下文类型，包含了工具运行所需的全部依赖：状态管理、中止控制、通知、文件缓存、权限上下文等。`ToolUseContext` 是工具与系统其余部分交互的主要桥梁。

通过 `buildTool` 工厂函数，工具开发者只需实现必要方法，其余"可默认"的方法（如 `isEnabled`、`checkPermissions`、`userFacingName` 等）由工厂统一填充，降低了新工具的开发负担，同时保证了默认的"失败关闭"安全策略。

---

## 核心内容详解

### 导入与依赖

| 模块 | 用途 |
|------|------|
| `@anthropic-ai/sdk` | ToolResultBlockParam、ToolUseBlockParam 类型 |
| `@modelcontextprotocol/sdk` | ElicitRequestURLParams、ElicitResult（MCP Elicitation） |
| `zod/v4` | 工具输入 schema 验证 |
| `./commands.js` | Command 类型 |
| `./hooks/useCanUseTool.js` | CanUseToolFn 权限检查函数类型 |
| `./types/permissions.js` | PermissionResult、PermissionMode 等 |
| `./types/tools.js` | 各工具进度数据类型（集中定义避免循环依赖） |
| `./state/AppState.js` | AppState 类型 |
| `./services/mcp/types.js` | MCPServerConnection、ServerResource |

### 主要类/函数/接口

- **名称**：`ToolUseContext`
  - **类型**：type（接口）
  - **用途**：工具执行时获取的完整上下文对象，包含工具运行所需的所有依赖
  - **关键字段**：
    - `options`：工具配置（命令列表、模型、MCP客户端等）
    - `abortController`：中止信号控制器
    - `readFileState`：文件状态LRU缓存
    - `getAppState` / `setAppState`：全局状态访问
    - `setAppStateForTasks`：专用于后台任务的状态更新（始终到达根存储）
    - `handleElicitation`：处理MCP -32042错误的URL Elicitation
    - `setToolJSX`：设置工具的React渲染内容
    - `messages`：当前会话消息列表
    - `contentReplacementState`：工具结果预算状态
    - `localDenialTracking`：异步子Agent的本地拒绝计数

- **名称**：`ToolPermissionContext`
  - **类型**：type（DeepImmutable 包装）
  - **用途**：描述当前权限配置，包括模式和规则集
  - **关键字段**：`mode`（default/auto/plan/bypassPermissions）、`alwaysAllowRules`、`alwaysDenyRules`、`alwaysAskRules`、`additionalWorkingDirectories`

- **名称**：`Tool<Input, Output, P>`
  - **类型**：泛型 type（接口）
  - **用途**：所有工具必须实现的完整接口定义
  - **核心方法**：
    - `call(args, context, canUseTool, parentMessage, onProgress?)` — 工具执行入口
    - `description(input, options)` — 生成工具描述（模型可见）
    - `checkPermissions(input, context)` — 工具级权限检查
    - `validateInput?(input, context)` — 输入合法性验证
    - `isConcurrencySafe(input)` — 是否可并发执行
    - `isReadOnly(input)` — 是否只读（影响权限提示）
    - `isDestructive?(input)` — 是否不可逆操作
    - `prompt(options)` — 生成工具的系统提示片段
    - `renderToolResultMessage?` — 渲染工具结果UI
    - `renderToolUseMessage` — 渲染工具调用UI（流式友好）
    - `renderGroupedToolUse?` — 批量工具调用分组渲染
    - `toAutoClassifierInput` — 用于安全分类器的紧凑表示
    - `backfillObservableInput?` — 向观察者公开的输入补全（幂等）
    - `mapToolResultToToolResultBlockParam` — 将结果序列化为API格式
    - `extractSearchText?` — 提取用于转录搜索的文本
  - **特殊字段**：
    - `shouldDefer?: boolean` — 是否延迟加载（需先用ToolSearch）
    - `alwaysLoad?: boolean` — 永不延迟，始终在首轮提示中可见
    - `mcpInfo?` — MCP工具的服务器和工具名（未规范化）
    - `maxResultSizeChars` — 超限时持久化到磁盘的阈值
    - `strict?` — 是否启用严格模式（参数schema强制执行）

- **名称**：`ToolDef<Input, Output, P>`
  - **类型**：type
  - **用途**：`buildTool` 接受的输入类型，可省略有默认值的方法
  - **关键逻辑**：`Omit<Tool, DefaultableToolKeys> & Partial<Pick<Tool, DefaultableToolKeys>>`

- **名称**：`buildTool<D>`
  - **类型**：function（泛型工厂）
  - **用途**：将工具定义补全为完整的 `Tool`，填入安全默认值
  - **参数**：`def: D extends AnyToolDef`
  - **返回值**：`BuiltTool<D>`（类型层面等价于 `{...TOOL_DEFAULTS, ...def}`）
  - **关键逻辑**：使用展开运算符合并 `TOOL_DEFAULTS` 和 `def`，类型层面通过 `BuiltTool<D>` 映射类型精确反映运行时行为

- **名称**：`TOOL_DEFAULTS`（内部常量）
  - **类型**：const 对象
  - **用途**：所有可默认方法的安全默认实现
  - **默认策略**（失败关闭）：
    - `isEnabled` → `true`
    - `isConcurrencySafe` → `false`（保守，假设不安全）
    - `isReadOnly` → `false`（假设有写操作）
    - `isDestructive` → `false`
    - `checkPermissions` → 允许（`{ behavior: 'allow', updatedInput }`）
    - `toAutoClassifierInput` → `''`（跳过分类器）
    - `userFacingName` → `name`

- **名称**：`toolMatchesName`
  - **类型**：function
  - **用途**：检查工具是否匹配给定名称（主名或别名）
  - **参数**：`tool: { name: string; aliases?: string[] }`，`name: string`
  - **返回值**：`boolean`

- **名称**：`findToolByName`
  - **类型**：function
  - **用途**：从工具列表中按名称或别名查找工具
  - **参数**：`tools: Tools`，`name: string`
  - **返回值**：`Tool | undefined`

- **名称**：`getEmptyToolPermissionContext`
  - **类型**：function
  - **用途**：创建空的权限上下文（用于测试或初始化）
  - **返回值**：`ToolPermissionContext`（所有规则为空，模式为 `'default'`）

- **名称**：`filterToolProgressMessages`
  - **类型**：function
  - **用途**：从进度消息中过滤出工具进度（排除 Hook 进度）
  - **参数**：`ProgressMessage[]`
  - **返回值**：`ProgressMessage<ToolProgressData>[]`

### 数据流与逻辑流程

```
用户输入 → 模型生成 tool_use →
  toolMatchesName() 查找工具 →
  validateInput() 验证输入 →
  canUseTool() / checkPermissions() 权限检查 →
  tool.call(args, ToolUseContext, ...) 执行 →
  onProgress() 实时进度更新 →
  返回 ToolResult<Output> →
  mapToolResultToToolResultBlockParam() 序列化 →
  renderToolResultMessage() 渲染UI
```

### 对外接口（导出内容）

主要导出：`Tool`、`Tools`、`ToolDef`、`ToolUseContext`、`ToolPermissionContext`、`ToolResult`、`ToolCallProgress`、`ToolProgress`、`ValidationResult`、`SetToolJSXFn`、`QueryChainTracking`、`CompactProgressEvent`、`buildTool`、`toolMatchesName`、`findToolByName`、`getEmptyToolPermissionContext`、`filterToolProgressMessages`、`AnyObject`。

同时重新导出来自 `./types/tools.js` 的各进度类型（向后兼容），以及来自 `./types/permissions.js` 的 `ToolPermissionRulesBySource`。

---

## 设计要点

- **集中定义避免循环依赖**：进度类型（`BashProgress` 等）和权限类型（`PermissionResult` 等）从集中位置导入后再重新导出，注释明确标注"break import cycles"。
- **泛型工厂 `buildTool` 的类型安全**：`BuiltTool<D>` 是一个精确的映射类型，在类型层面模拟 `{...defaults, ...def}` 的运行时行为，避免了 `satisfies Tool` 写法导致的字面量类型丢失问题。
- **失败关闭原则**：默认权限是允许（委托给通用权限系统），但 `isConcurrencySafe` 默认为 `false`、`isReadOnly` 默认为 `false`，倾向于安全保守的行为。
- **延迟加载支持**：`shouldDefer` 和 `alwaysLoad` 字段支持工具的按需加载（ToolSearch 机制），减少首轮提示的token用量。
- **渲染层分离**：工具负责自己的UI渲染（多个 `render*` 方法），但渲染是可选的，omit 则使用降级渲染。

---

## 与其他文件的关系

- **依赖**：
  - `src/types/permissions.ts`、`src/types/tools.ts`（集中类型定义）
  - `src/state/AppState.ts`（AppState）
  - `src/services/mcp/types.ts`（MCPServerConnection）
  - `src/hooks/useCanUseTool.ts`（CanUseToolFn）
- **被依赖**：
  - 所有工具实现（`src/tools/` 下的每个工具）
  - `src/QueryEngine.ts`（ToolUseContext 组装）
  - `src/query.ts`（工具执行循环）
  - UI 组件（使用 render* 方法）

---

## 注意事项

- `ToolUseContext.setAppState` 在异步子Agent中是 no-op；需要跨Agent生命周期的状态更新必须使用 `setAppStateForTasks`。
- `Tool.outputSchema` 目前是可选的（注释标注"TungstenTool不定义这个"），未来计划改为必须。
- `backfillObservableInput` 必须是幂等的，且不能修改原始 API 输入（保护 prompt cache）。
- `maxResultSizeChars` 设为 `Infinity` 的工具（如 `Read`）永不持久化结果，避免循环引用。
- `renderToolUseMessage` 的 `input` 参数是 `Partial`，因为在工具参数流式传输完成前就会调用渲染。
