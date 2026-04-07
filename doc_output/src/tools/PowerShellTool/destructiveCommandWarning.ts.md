# destructiveCommandWarning.ts — 破坏性命令警告

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/destructiveCommandWarning.ts`
- **作用**: 检测潜在的破坏性PowerShell命令并生成警告

## 核心内容详解

### 破坏性模式

```typescript
const DESTRUCTIVE_PATTERNS: DestructivePattern[] = [
  // Remove-Item递归强制删除
  {
    pattern: /(?:^|[|;&\n({])\s*(Remove-Item|rm|del|rd|rmdir|ri)\b[^|;&\n}]*-Recurse\b[^|;&\n}]*-Force\b/i,
    warning: 'Note: may recursively force-remove files',
  },
  
  // Clear-Content通配符
  { pattern: /\bClear-Content\b[^|;&\n]*\*/i, warning: 'Note: may clear content of multiple files' },
  
  // Format-Volume
  { pattern: /\bFormat-Volume\b/i, warning: 'Note: may format a disk volume' },
  
  // Clear-Disk
  { pattern: /\bClear-Disk\b/i, warning: 'Note: may clear a disk' },
  
  // Git破坏性操作
  { pattern: /\bgit\s+reset\s+--hard\b/i, warning: 'Note: may discard uncommitted changes' },
  { pattern: /\bgit\s+push\b[^|;&\n]*\s+(--force|--force-with-lease|-f)\b/i, warning: 'Note: may overwrite remote history' },
  { pattern: /\bgit\s+clean\b(?![^|;&\n]*(?:-[a-zA-Z]*n|--dry-run))[^|;&\n]*-[a-zA-Z]*f/i, warning: 'Note: may permanently delete untracked files' },
  
  // 数据库操作
  { pattern: /\b(DROP|TRUNCATE)\s+(TABLE|DATABASE|SCHEMA)\b/i, warning: 'Note: may drop or truncate database objects' },
  
  // 系统操作
  { pattern: /\bStop-Computer\b/i, warning: 'Note: will shut down the computer' },
  { pattern: /\bRestart-Computer\b/i, warning: 'Note: will restart the computer' },
]
```

### 核心函数

```typescript
export function getDestructiveCommandWarning(command: string): string | null
```

- 返回警告字符串或null
- 纯粹信息性，不影响权限逻辑

## 设计要点

1. **锚定模式**: 使用`^`、`|`、`;`等锚定到语句开始
2. **避免误报**: `git rm --force`不应匹配rm模式
3. **非阻塞**: 仅生成警告，不阻止执行

## 与其他文件的关系

- **permissions.ts**: 调用生成警告文本
