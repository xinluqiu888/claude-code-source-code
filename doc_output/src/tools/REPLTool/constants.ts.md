# constants.ts — REPL工具常量

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/REPLTool/constants.ts`
- **作用**: 定义REPL工具名称和模式检测常量

## 核心内容详解

### 工具名称常量

```typescript
export const REPL_TOOL_NAME = 'REPL'
```

### REPL模式检测

```typescript
export function isReplModeEnabled(): boolean
```

检测逻辑：
1. 如果`CLAUDE_CODE_REPL`设置为falsy，返回false
2. 如果`CLAUDE_REPL_MODE`为truthy，返回true
3. 默认：当`USER_TYPE`为'ant'且`CLAUDE_CODE_ENTRYPOINT`为'cli'时返回true

**设计考虑**: SDK入口点（sdk-ts, sdk-py, sdk-cli）默认不启用REPL模式，因为SDK用户直接脚本化工具调用（Bash、Read等），而REPL模式会隐藏这些工具。

### REPL专属工具列表

```typescript
export const REPL_ONLY_TOOLS = new Set([
  FILE_READ_TOOL_NAME,
  FILE_WRITE_TOOL_NAME,
  FILE_EDIT_TOOL_NAME,
  GLOB_TOOL_NAME,
  GREP_TOOL_NAME,
  BASH_TOOL_NAME,
  NOTEBOOK_EDIT_TOOL_NAME,
  AGENT_TOOL_NAME,
])
```

当REPL模式启用时，这些工具从Claude的直接使用中隐藏，强制Claude使用REPL进行批量操作。

## 设计要点

1. **环境变量控制**: 支持多种方式控制REPL模式开关
2. **SDK保护**: SDK用户不受REPL模式影响
3. **工具隐藏**: 核心工具在REPL模式下隐藏

## 与其他文件的关系

- **其他工具常量文件**: 导入各工具的常量名称
- **primitiveTools.ts**: 使用REPL_ONLY_TOOLS集合
