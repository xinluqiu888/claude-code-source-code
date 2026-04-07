# modeValidation.ts — 权限模式验证

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/modeValidation.ts`
- **作用**: 根据当前权限模式验证PowerShell命令

## 核心内容详解

### acceptEdits模式允许列表

```typescript
const ACCEPT_EDITS_ALLOWED_CMDLETS = new Set([
  'set-content',
  'add-content',
  'remove-item',
  'clear-content',
])
```

### 符号链接检测

```typescript
const LINK_ITEM_TYPES = new Set(['symboliclink', 'junction', 'hardlink'])

export function isSymlinkCreatingCommand(cmd: { name: string; args: string[] }): boolean
```

检测`New-Item -ItemType SymbolicLink/Junction/HardLink`。

### 核心函数: checkPermissionMode

```typescript
export function checkPermissionMode(
  input: { command: string },
  parsed: ParsedPowerShellCommand,
  toolPermissionContext: ToolPermissionContext,
): PermissionResult
```

#### 安全检查

1. **安全标志检查**: 拒绝包含子表达式、脚本块、成员调用的命令
2. **复合命令cwd desync防护**: 如果复合命令包含目录更改命令+写入命令，拒绝
3. **链接创建防护**: 如果复合命令创建符号链接，拒绝
4. **元素类型白名单**: 只允许StringConstant和Parameter
5. **参数验证**: 检查参数值是否可静态解析

#### 复合命令检查示例

```powershell
# 会被拒绝：Set-Location后写入
Set-Location ./.claude; Set-Content ./settings.json '...'

# 会被拒绝：创建链接后读取
New-Item -ItemType SymbolicLink -Path ./link -Value /etc; Get-Content ./link/passwd
```

## 设计要点

1. **AST分析**: 基于解析树而非正则表达式
2. **TOCTOU防护**: 防范验证和执行时路径解析不一致
3. **保守策略**: 不确定时回退到'ask'

## 与其他文件的关系

- **powershellPermissions.ts**: 调用进行模式检查
- **readOnlyValidation.ts**: 共享解析工具
