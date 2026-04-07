# toolName.ts — PowerShell工具名称

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/toolName.ts`
- **作用**: 定义PowerShell工具名称常量

## 核心内容详解

```typescript
// 单独文件以打破与prompt.ts的循环依赖
export const POWERSHELL_TOOL_NAME = 'PowerShell' as const
```

## 设计要点

- 单独文件避免循环依赖
- 被prompt.ts和PowerShellTool.tsx共同使用

## 与其他文件的关系

- **prompt.ts**: 导入工具名称
- **PowerShellTool.tsx**: 导入工具名称
