# commandSemantics.ts — 命令退出码语义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/commandSemantics.ts`
- **作用**: 解释PowerShell命令的退出码语义

## 核心内容详解

### 背景说明

PowerShell cmdlet使用终止错误（`$?`）而非退出码表示失败，但外部可执行文件设置`$LASTEXITCODE`：

- **grep/rg**: 退出码1表示无匹配（非错误）
- **robocopy**: 退出码0-7表示成功

### 默认语义

```typescript
const DEFAULT_SEMANTIC: CommandSemantic = (exitCode) => ({
  isError: exitCode !== 0,
  message: exitCode !== 0 ? `Command failed with exit code ${exitCode}` : undefined,
})
```

### 命令特定语义

```typescript
const COMMAND_SEMANTICS: Map<string, CommandSemantic> = new Map([
  // grep / ripgrep: 0=匹配, 1=无匹配, 2+=错误
  ['grep', GREP_SEMANTIC],
  ['rg', GREP_SEMANTIC],
  
  // findstr: 同上
  ['findstr', GREP_SEMANTIC],
  
  // robocopy: 位字段语义
  // 0 = 无文件复制（已同步）
  // 1 = 文件复制成功
  // 2 = 检测到额外文件/目录
  // 4 = 检测到不匹配文件/目录
  // 8 = 某些文件/目录无法复制（错误）
  // 16 = 严重错误
  ['robocopy', (exitCode) => ({
    isError: exitCode >= 8,
    message: exitCode === 0 ? 'No files copied (already in sync)'
      : exitCode >= 1 && exitCode < 8 ? 'Files copied successfully'
      : undefined,
  })],
])
```

### 命令提取

```typescript
function heuristicallyExtractBaseCommand(command: string): string
```

- 分割管道符`|`
- 取最后一段（决定退出码的那段）
- 提取基本命令名（无路径、无.exe后缀）

## 设计要点

1. **精确匹配**: 基于命令名的小写匹配
2. **管道感知**: 取最后一段作为退出码来源
3. **保守策略**: 未知命令使用默认语义（非0即错误）

## 与其他文件的关系

- **PowerShellTool.tsx**: 使用interpretCommandResult解释退出码
