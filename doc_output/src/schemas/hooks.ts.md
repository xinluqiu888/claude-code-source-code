# hooks.ts — Hook Schema 定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/schemas/hooks.ts`
- **类型**: TypeScript 模块
- **导出内容**: HookCommandSchema、HookMatcherSchema、HooksSchema 及派生类型
- **依赖关系**:
  - 导入: `agentSdkTypes.js`, `zod/v4`, `lazySchema.js`, `shell/shellProvider.js`

## 功能概述

本文件包含 Hook 相关的 Zod Schema 定义，专门提取到这个位置以打破 import 循环。原始定义在 `src/utils/settings/types.ts`，通过提取到共享位置，settings/types.ts 和 plugins/schemas.ts 都可以从这里导入，避免相互导入。

## 核心内容详解

### 1. IfConditionSchema (第19-27行)

共享的 `if` 条件字段 Schema：

```typescript
const IfConditionSchema = lazySchema(() =>
  z
    .string()
    .optional()
    .describe(
      'Permission rule syntax to filter when this hook runs (e.g., "Bash(git *)"). ' +
        'Only runs if the tool call matches the pattern. Avoids spawning hooks for non-matching commands.',
    ),
)
```

**用途**: 使用权限规则语法（如 "Bash(git *)"）在 Hook 执行前进行过滤，避免为不匹配的命令生成 Hook。

### 2. buildHookSchemas() 工厂函数 (第31-171行)

构建各种 Hook Schema 的内部工厂函数：

#### BashCommandHookSchema (第32-65行)

Shell 命令 Hook：

```typescript
const BashCommandHookSchema = z.object({
  type: z.literal('command').describe('Shell command hook type'),
  command: z.string().describe('Shell command to execute'),
  if: IfConditionSchema(),
  shell: z
    .enum(SHELL_TYPES)
    .optional()
    .describe("Shell interpreter. 'bash' uses your $SHELL (bash/zsh/sh); 'powershell' uses pwsh. Defaults to bash."),
  timeout: z.number().positive().optional().describe('Timeout in seconds for this specific command'),
  statusMessage: z.string().optional().describe('Custom status message to display in spinner while hook runs'),
  once: z.boolean().optional().describe('If true, hook runs once and is removed after execution'),
  async: z.boolean().optional().describe('If true, hook runs in background without blocking'),
  asyncRewake: z.boolean().optional().describe(
    'If true, hook runs in background and wakes the model on exit code 2 (blocking error). Implies async.',
  ),
})
```

**特有字段**:
- `command`: 要执行的 shell 命令
- `shell`: 解释器类型（bash/powershell）
- `async`: 后台运行不阻塞
- `asyncRewake`: 后台运行并在退出码 2 时唤醒模型

#### PromptHookSchema (第67-95行)

LLM 提示 Hook：

```typescript
const PromptHookSchema = z.object({
  type: z.literal('prompt').describe('LLM prompt hook type'),
  prompt: z.string().describe('Prompt to evaluate with LLM. Use $ARGUMENTS placeholder for hook input JSON.'),
  if: IfConditionSchema(),
  timeout: z.number().positive().optional().describe('Timeout in seconds for this specific prompt evaluation'),
  model: z.string().optional().describe('Model to use for this prompt hook (e.g., "claude-sonnet-4-6")'),
  statusMessage: z.string().optional().describe('Custom status message to display in spinner while hook runs'),
  once: z.boolean().optional().describe('If true, hook runs once and is removed after execution'),
})
```

**特有字段**:
- `prompt`: 使用 `$ARGUMENTS` 占位符的提示模板
- `model`: 可选指定模型（如 "claude-sonnet-4-6"）

#### HttpHookSchema (第97-126行)

HTTP Hook：

```typescript
const HttpHookSchema = z.object({
  type: z.literal('http').describe('HTTP hook type'),
  url: z.string().url().describe('URL to POST the hook input JSON to'),
  if: IfConditionSchema(),
  timeout: z.number().positive().optional().describe('Timeout in seconds for this specific request'),
  headers: z.record(z.string(), z.string()).optional().describe(
    'Additional headers to include in the request. Values may reference environment variables...',
  ),
  allowedEnvVars: z.array(z.string()).optional().describe(
    'Explicit list of environment variable names that may be interpolated in header values...',
  ),
  statusMessage: z.string().optional().describe('Custom status message to display in spinner while hook runs'),
  once: z.boolean().optional().describe('If true, hook runs once and is removed after execution'),
})
```

**特有字段**:
- `url`: POST Hook 输入 JSON 的目标 URL
- `headers`: 支持环境变量引用的请求头
- `allowedEnvVars`: 允许在请求头中解析的环境变量白名单

#### AgentHookSchema (第128-163行)

代理验证器 Hook：

```typescript
const AgentHookSchema = z.object({
  type: z.literal('agent').describe('Agentic verifier hook type'),
  prompt: z.string().describe(
    'Prompt describing what to verify (e.g. "Verify that unit tests ran and passed."). Use $ARGUMENTS placeholder for hook input JSON.',
  ),
  if: IfConditionSchema(),
  timeout: z.number().positive().optional().describe('Timeout in seconds for agent execution (default 60)'),
  model: z.string().optional().describe('Model to use for this agent hook (e.g., "claude-sonnet-4-6")'),
  statusMessage: z.string().optional().describe('Custom status message to display in spinner while hook runs'),
  once: z.boolean().optional().describe('If true, hook runs once and is removed after execution'),
})
```

**重要说明** (第131-137行):
- **不要在这里添加 .transform()**
- 此 schema 用于 parseSettingsFile
- updateSettingsForSource 会将解析结果通过 JSON.stringify 往返
- transform 的函数值会被静默删除，导致用户的 prompt 从 settings.json 中丢失
- 原始 transform 用于程序化构造场景，已重构到 VerifyPlanExecutionTool

### 3. HookCommandSchema (第176-189行)

Hook 命令 Schema（不包括函数 Hook，它们无法持久化）：

```typescript
export const HookCommandSchema = lazySchema(() => {
  const {
    BashCommandHookSchema,
    PromptHookSchema,
    AgentHookSchema,
    HttpHookSchema,
  } = buildHookSchemas()
  return z.discriminatedUnion('type', [
    BashCommandHookSchema,
    PromptHookSchema,
    AgentHookSchema,
    HttpHookSchema,
  ])
})
```

使用 `discriminatedUnion` 基于 `type` 字段区分四种 Hook 类型。

### 4. HookMatcherSchema (第194-204行)

匹配器配置 Schema：

```typescript
export const HookMatcherSchema = lazySchema(() =>
  z.object({
    matcher: z.string().optional().describe('String pattern to match (e.g. tool names like "Write")'),
    hooks: z.array(HookCommandSchema()).describe('List of hooks to execute when the matcher matches'),
  }),
)
```

**字段**:
- `matcher`: 字符串匹配模式（如工具名 "Write"）
- `hooks`: 匹配时执行的 Hook 列表

### 5. HooksSchema (第207-213行)

Hooks 配置 Schema：

```typescript
export const HooksSchema = lazySchema(() =>
  z.partialRecord(z.enum(HOOK_EVENTS), z.array(HookMatcherSchema())),
)
```

**特点**:
- 使用 `partialRecord`（部分记录）
- 键是 HOOK_EVENTS 枚举
- 值是 HookMatcher 数组
- 不需要定义所有 Hook 事件

### 6. 派生类型 (第215-222行)

从 Schema 推断的 TypeScript 类型：

```typescript
export type HookCommand = z.infer<ReturnType<typeof HookCommandSchema>>
export type BashCommandHook = Extract<HookCommand, { type: 'command' }>
export type PromptHook = Extract<HookCommand, { type: 'prompt' }>
export type AgentHook = Extract<HookCommand, { type: 'agent' }>
export type HttpHook = Extract<HookCommand, { type: 'http' }>
export type HookMatcher = z.infer<ReturnType<typeof HookMatcherSchema>>
export type HooksSettings = Partial<Record<HookEvent, HookMatcher[]>>
```

## 设计要点

1. **懒加载 Schema**: 所有 Schema 使用 `lazySchema` 包装，避免循环依赖
2. **类型安全**: 使用 `z.discriminatedUnion` 确保类型安全
3. **详细文档**: 每个字段都有 `.describe()` 说明
4. **条件过滤**: 支持 `if` 条件避免不必要的 Hook 执行
5. **无 transform**: 由于序列化问题，避免使用 Zod transform

## 与其他文件的关系

- **utils/settings/types.ts**: 原始位置，现在从这里导入
- **utils/plugins/schemas.ts**: 从这个共享位置导入
- **utils/shell/shellProvider.ts**: 提供 SHELL_TYPES

## 配置示例

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Running bash command'",
            "if": "Bash(git *)",
            "timeout": 30
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Welcome the user to the session.",
            "model": "claude-sonnet-4-6"
          }
        ]
      }
    ]
  }
}
```

## 注意事项

1. **循环依赖**: 文件位置专门设计用于打破 settings/types.ts 和 plugins/schemas.ts 之间的循环
2. **无函数 Hook**: HookCommandSchema 只包含可持久化的 Hook 类型
3. **环境变量**: HttpHook 的 headers 支持 `$VAR_NAME` 或 `${VAR_NAME}` 语法
4. **Transform 禁止**: 不要添加 .transform()，会导致设置丢失
