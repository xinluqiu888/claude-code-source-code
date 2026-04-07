# readOnlyValidation.ts — 只读命令验证

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/readOnlyValidation.ts`
- **作用**: 验证PowerShell命令是否为只读操作

## 核心内容详解

### 只读Cmdlet列表

```typescript
const READONLY_CMDLETS = new Set([
  // 文件系统读取
  'get-content', 'gc', 'cat', 'type',
  'get-childitem', 'gci', 'ls', 'dir',
  'get-item', 'gi',
  'test-path',
  'resolve-path',
  'split-path',
  // ...
])
```

### 安全输出Cmdlet

```typescript
const SAFE_OUTPUT_CMDLETS = new Set([
  'out-null',
  'out-default',
  'out-string',
  'out-host',
  'out-file', // 验证过路径后
  // ...
])
```

### 管道尾部允许列表

```typescript
const CMDLET_ALLOWLIST = new Map<string, CmdletAllowlistEntry>([
  ['format-table', { maxArgCount: 0 }],
  ['format-list', { maxArgCount: 0 }],
  ['select-object', { maxArgCount: 2 }],
  // ...
])
```

### 路径解析

```typescript
export function resolveToCanonical(name: string): string
```

将别名解析为规范cmdlet名称。

## 设计要点

1. **别名处理**: 支持常见别名
2. **参数验证**: 确保参数不会泄漏值
3. **管道感知**: 允许管道尾部转换器
4. **保守策略**: 不确定时要求确认

## 与其他文件的关系

- **powershellPermissions.ts**: 被调用验证只读操作
- **commonParameters.ts**: 通用参数处理
