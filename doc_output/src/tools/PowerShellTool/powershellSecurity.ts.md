# powershellSecurity.ts — PowerShell安全分析

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/powershellSecurity.ts`
- **作用**: PowerShell命令的AST级安全分析

## 核心内容详解

### 安全检查器列表

```typescript
const validators = [
  checkInvokeExpression,      // Invoke-Expression / iex
  checkDynamicCommandName,    // 动态命令名
  checkEncodedCommand,        // 编码命令 -e
  checkPwshCommandOrFile,     // 嵌套PowerShell
  checkDownloadCradles,       // 下载cradle (IWR | IEX)
  checkDownloadUtilities,     // 下载工具 (BitsTransfer)
  checkAddType,               // Add-Type编译.NET
  checkComObject,             // COM对象
  checkDangerousFilePathExecution, // -FilePath执行
  checkInvokeItem,            // Invoke-Item (ShellExecute)
  checkScheduledTask,         // 计划任务
  checkForEachMemberName,     // ForEach-Object -MemberName
  checkStartProcess,          // Start-Process特权提升
  checkScriptBlockInjection,  // 脚本块注入
  checkSubExpressions,        // 子表达式 $()
  checkExpandableStrings,     // 可展开字符串 "$var"
  checkSplatting,             // Splatting @args
  checkStopParsing,           // --% 停止解析
  checkMemberInvocations,     // .NET方法调用
  checkTypeLiterals,          // 类型字面量 [Type]
  checkEnvVarManipulation,    // 环境变量操作
  checkModuleLoading,         // 模块加载
  checkRuntimeStateManipulation, // 运行时状态操作
  checkWmiProcessSpawn,       // WMI进程创建
]
```

### 关键检查示例

#### 下载Cradle检测

```typescript
function checkDownloadCradles(parsed): PowerShellSecurityResult {
  // 检测: Invoke-WebRequest ... | Invoke-Expression
  const DOWNLOADER_NAMES = new Set([
    'invoke-webrequest', 'iwr', 'invoke-restmethod', 'irm', 'new-object'
  ])
}
```

#### COM对象检测

```typescript
function checkComObject(parsed): PowerShellSecurityResult {
  // 检测: New-Object -ComObject WScript.Shell
}
```

#### 类型字面量检测

```typescript
function checkTypeLiterals(parsed): PowerShellSecurityResult {
  for (const t of parsed.typeLiterals ?? []) {
    if (!isClmAllowedType(t)) {
      return { behavior: 'ask', message: `类型[${t}]不在CLM白名单` }
    }
  }
}
```

## 设计要点

1. **AST分析**: 基于PowerShell解析树，而非正则表达式
2. **深度检查**: 递归检查嵌套命令
3. **上下文感知**: 区分安全和不安全的用法
4. **CLM集成**: 验证类型是否在Constrained Language Mode允许列表

## 与其他文件的关系

- **powershellPermissions.ts**: 被调用进行安全检查
- **clmTypes.ts**: 类型白名单验证
