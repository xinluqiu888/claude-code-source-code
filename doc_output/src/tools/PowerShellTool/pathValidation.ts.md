# pathValidation.ts — 路径验证

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/pathValidation.ts`
- **作用**: PowerShell命令的路径约束验证

## 核心内容详解

### 路径配置

```typescript
const CMDLET_PATH_CONFIG: ReadonlyMap<string, PathConstraintConfig> = new Map([
  ['set-content', { ... }],
  ['add-content', { ... }],
  ['remove-item', { ... }],
  ['clear-content', { ... }],
  // ...
])
```

### 验证流程

```typescript
export function checkPathConstraints(
  input: { command: string },
  parsed: ParsedPowerShellCommand,
  context: ToolPermissionContext,
): CheckPathConstraintsResult
```

1. **解析命令**: 分割管道段，提取命令和参数
2. **检查每段命令**:
   - 路径参数值验证
   - 危险文件检测
   - 危险目录检测
   - 绝对路径检测
3. **返回结果**: allow / ask / deny

### 路径类型分类

- **Literal路径**: 无通配符，字面量路径
- **Provider路径**: 如`env:PATH`、`HKLM:\Software`
- **通配符路径**: 包含`*`、`?`、`[]`

## 设计要点

1. **细粒度控制**: 每个cmdlet有自己的路径约束
2. **多层验证**: 字面量、provider、通配符分别处理
3. **Git安全集成**: 检测Git内部路径访问

## 与其他文件的关系

- **powershellPermissions.ts**: 调用进行路径验证
- **gitSafety.ts**: 检测Git路径
