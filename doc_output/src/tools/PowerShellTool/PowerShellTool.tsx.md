# PowerShellTool.tsx — PowerShell执行工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/PowerShellTool.tsx`
- **工具名称**: `PowerShell`

## 功能概述

在Windows系统上执行PowerShell命令。支持PowerShell 5.1 (Windows PowerShell)和PowerShell 7+ (PowerShell Core)。

## 核心内容详解

### 主要导入

```typescript
import { commandSemantics } from './commandSemantics.js'
import { clmTypes } from './clmTypes.js'
import { commonParameters } from './commonParameters.js'
import { destructiveCommandWarning } from './destructiveCommandWarning.js'
import { gitSafety } from './gitSafety.js'
import { modeValidation } from './modeValidation.js'
import { pathValidation } from './pathValidation.js'
import { powershellPermissions } from './powershellPermissions.js'
import { powershellSecurity } from './powershellSecurity.js'
import { readOnlyValidation } from './readOnlyValidation.js'
```

### 安全模块

PowerShellTool包含多个安全验证模块：

1. **powershellSecurity.ts**: AST级安全检查
2. **pathValidation.ts**: 路径验证
3. **modeValidation.ts**: 权限模式验证
4. **readOnlyValidation.ts**: 只读命令验证
5. **powershellPermissions.ts**: 权限检查主入口
6. **destructiveCommandWarning.ts**: 破坏性命令警告

### 版本适配

```typescript
// 检测PowerShell版本
const edition = await getPowerShellEdition()

// Desktop (5.1)
// - 不支持 && || 管道链操作符
// - 不支持 ?. ?? ?: 操作符
// - 默认UTF-16 LE编码
// - stderr重定向会产生ErrorRecord

// Core (7+)
// - 支持所有现代语法
// - 默认UTF-8无BOM
```

### 命令语义

```typescript
// 处理不同命令的退出码语义
export function interpretCommandResult(
  command: string,
  exitCode: number,
  stdout: string,
  stderr: string,
): { isError: boolean; message?: string }
```

特殊处理：
- **grep/rg**: 退出码1表示无匹配（非错误）
- **findstr**: 退出码1表示无匹配
- **robocopy**: 退出码0-7表示成功

## 设计要点

1. **多版本支持**: 自动检测并适配PS 5.1和7+
2. **深度安全分析**: AST级解析和验证
3. **CLM类型检查**: 基于Constrained Language Mode的类型白名单
4. **Git安全**: 防止通过Git命令进行沙箱逃逸

## 与其他文件的关系

- **prompt.ts**: 动态生成版本特定的提示
- **UI.tsx**: 渲染函数
- **toolName.ts**: 工具名称常量

## 注意事项

- 命令在-NonInteractive模式下运行
- 支持run_in_background后台执行
- 工作目录在命令间保持
- shell状态（变量、函数）不保持
