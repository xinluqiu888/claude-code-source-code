# command.ts — 命令系统类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/types/command.ts`
- **类型**: TypeScript 模块
- **导出内容**: `Command` 类型、`PromptCommand` 类型、`LocalCommand` 类型、`LocalJSXCommand` 类型、`CommandBase` 类型等，以及相关工具函数
- **依赖关系**:
  - 导入: `@anthropic-ai/sdk`, `crypto`, `useCanUseTool.js`, `compact.js`, `mcp/types.js`, `Tool.js`, `effort.js`, `ide.js`, `settings/constants.js`, `settings/types.js`, `theme.js`, `logs.js`, `message.js`, `plugin.js`

## 功能概述

本文件定义了 Claude Code 命令系统的完整类型体系，包括三种主要命令类型：提示命令（PromptCommand）、本地命令（LocalCommand）和本地 JSX 命令（LocalJSXCommand）。它还定义了命令的元数据、可用性控制、结果类型和执行上下文，是命令系统的类型基础。

## 核心内容详解

### 1. LocalCommandResult 类型 (第16-23行)

本地命令执行结果的联合类型：

```typescript
export type LocalCommandResult =
  | { type: 'text'; value: string }                    // 纯文本结果
  | { type: 'compact'; compactionResult: CompactionResult; displayText?: string }  // 压缩结果
  | { type: 'skip' }                                   // 跳过消息
```

### 2. PromptCommand 类型 (第25-57行)

基于 LLM 提示的命令类型，通过生成提示内容执行：

**核心字段**:
- `type: 'prompt'`: 类型标识
- `progressMessage`: 执行时显示的进度消息
- `contentLength`: 命令内容长度（用于 token 估算）
- `allowedTools`: 允许的工具白名单
- `model`: 可选指定模型
- `source`: 来源（settings, builtin, mcp, plugin, bundled）
- `disableNonInteractive`: 是否禁用非交互模式
- `hooks`: 命令触发时注册的钩子
- `context`: 执行上下文（'inline' 或 'fork'）
- `agent`: fork 模式下使用的代理类型
- `effort`: 工作量配置
- `paths`: 适用的文件路径 glob 模式
- `getPromptForCommand()`: 异步获取提示内容

### 3. LocalCommand & LocalCommandModule (第62-78行)

本地 JavaScript 函数命令：

```typescript
export type LocalCommandCall = (
  args: string,
  context: LocalJSXCommandContext,
) => Promise<LocalCommandResult>

export type LocalCommandModule = {
  call: LocalCommandCall
}

type LocalCommand = {
  type: 'local'
  supportsNonInteractive: boolean
  load: () => Promise<LocalCommandModule>
}
```

### 4. LocalJSXCommand & LocalJSXCommandModule (第80-152行)

基于 React/JSX 的交互式命令：

**执行上下文**:
```typescript
export type LocalJSXCommandContext = ToolUseContext & {
  canUseTool?: CanUseToolFn
  setMessages: (updater: (prev: Message[]) => Message[]) => void
  options: {
    dynamicMcpConfig?: Record<string, ScopedMcpServerConfig>
    ideInstallationStatus: IDEExtensionInstallationStatus | null
    theme: ThemeName
  }
  onChangeAPIKey: () => void
  onChangeDynamicMcpConfig?: (config: Record<string, ScopedMcpServerConfig>) => void
  onInstallIDEExtension?: (ide: IdeType) => void
  resume?: (sessionId: UUID, log: LogOption, entrypoint: ResumeEntrypoint) => Promise<void>
}
```

**执行签名**:
```typescript
export type LocalJSXCommandCall = (
  onDone: LocalJSXCommandOnDone,
  context: ToolUseContext & LocalJSXCommandContext,
  args: string,
) => Promise<React.ReactNode>
```

### 5. CommandAvailability (第155-173行)

命令可用性声明，控制谁可以使用该命令：

```typescript
export type CommandAvailability =
  | 'claude-ai'    // Claude.ai OAuth 订阅用户
  | 'console'      // Console API key 用户
```

**设计原则**: 与 `isEnabled()` 分离：
- `availability`: 谁可以使用（静态，基于认证）
- `isEnabled()`: 当前是否启用（动态，基于特性标志）

### 6. CommandBase (第175-203行)

所有命令共有的基础字段：

| 字段 | 说明 |
|------|------|
| `availability` | 可用性限制 |
| `description` | 命令描述 |
| `isEnabled` | 是否启用（动态） |
| `isHidden` | 是否隐藏（不显示在帮助中） |
| `name` | 命令名称 |
| `aliases` | 别名数组 |
| `isMcp` | 是否来自 MCP |
| `argumentHint` | 参数提示文本 |
| `whenToUse` | 使用场景说明 |
| `version` | 版本 |
| `disableModelInvocation` | 禁止模型调用 |
| `userInvocable` | 用户是否可调起 |
| `loadedFrom` | 加载来源 |
| `kind` | 类型（如 'workflow'） |
| `immediate` | 是否立即执行（绕过队列） |
| `isSensitive` | 是否敏感（参数脱敏） |
| `userFacingName` | 用户可见名称 |

### 7. Command 类型 (第205-206行)

命令的完整类型，是三种命令类型的联合：

```typescript
export type Command = CommandBase &
  (PromptCommand | LocalCommand | LocalJSXCommand)
```

### 8. 工具函数 (第208-216行)

**getCommandName(cmd)**: 获取用户可见的命令名称
```typescript
export function getCommandName(cmd: CommandBase): string {
  return cmd.userFacingName?.() ?? cmd.name
}
```

**isCommandEnabled(cmd)**: 检查命令是否启用
```typescript
export function isCommandEnabled(cmd: CommandBase): boolean {
  return cmd.isEnabled?.() ?? true
}
```

## 设计要点

1. **联合类型**: 使用 TypeScript 联合类型区分不同命令类型
2. **延迟加载**: 本地命令使用 `load()` 函数延迟加载实现
3. **类型安全**: 完整的上下文和结果类型确保编译时安全
4. **可扩展性**: 通过 CommandBase 可轻松添加新的命令类型
5. **权限控制**: 双层权限（availability + isEnabled）

## 与其他文件的关系

- **commands.ts**: 实现命令注册和执行
- **Tool.ts**: LocalJSXCommandContext 依赖 ToolUseContext
- **settings/types.ts**: 使用 HooksSettings 类型
- **mcp/types.ts**: 使用 MCP 相关类型

## 使用场景

```typescript
// 定义一个提示命令
const myPromptCommand: Command = {
  type: 'prompt',
  name: 'my-command',
  description: 'My custom command',
  source: 'plugin',
  progressMessage: 'Executing...',
  getPromptForCommand: async (args, context) => {
    return [{ type: 'text', text: `Process: ${args}` }]
  }
}

// 定义一个本地 JSX 命令
const myJSXCommand: Command = {
  type: 'local-jsx',
  name: 'my-ui',
  description: 'Interactive UI',
  load: async () => ({
    call: async (onDone, context, args) => {
      return <MyComponent onDone={onDone} />
    }
  })
}
```
