# prompt.ts — PowerShell提示生成

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/prompt.ts`
- **作用**: 动态生成PowerShell版本特定的提示

## 核心内容详解

### 版本特定指导

```typescript
function getEditionSection(edition: PowerShellEdition | null): string {
  if (edition === 'desktop') {
    // Windows PowerShell 5.1
    return `PowerShell edition: Windows PowerShell 5.1 (powershell.exe)
   - Pipeline chain operators \`&&\` and \`||\` are NOT available
   - Ternary (\`?:\`), null-coalescing (\`??\`), null-conditional (\`?.\`) NOT available
   - Avoid \`2>&1\` on native executables (ErrorRecord wrapping)
   - Default file encoding is UTF-16 LE (with BOM)
   - ConvertFrom-Json returns PSCustomObject (no -AsHashtable)`
  }
  if (edition === 'core') {
    // PowerShell 7+
    return `PowerShell edition: PowerShell 7+ (pwsh)
   - Pipeline chain operators \`&&\` and \`||\` ARE available
   - Ternary, null-coalescing, null-conditional operators available
   - Default file encoding is UTF-8 without BOM`
  }
  // 未知版本 - 保守假设5.1
}
```

### 动态内容

```typescript
export async function getPrompt(): Promise<string> {
  const backgroundNote = getBackgroundUsageNote()
  const sleepGuidance = getSleepGuidance()
  const edition = await getPowerShellEdition()
  
  return `Executes a given PowerShell command...
${getEditionSection(edition)}
...`
}
```

### 背景任务指导

```typescript
function getBackgroundUsageNote(): string | null {
  if (isEnvTruthy(process.env.CLAUDE_CODE_DISABLE_BACKGROUND_TASKS)) {
    return null
  }
  return `  - You can use the \`run_in_background\` parameter...`
}
```

### 睡眠指导

```typescript
function getSleepGuidance(): string | null {
  return `  - Avoid unnecessary \`Start-Sleep\` commands:
    - Do not sleep between commands that can run immediately
    - If your command is long running... use \`run_in_background\`
    - Do not retry failing commands in a sleep loop
    - If waiting for a background task... do not poll
    - If you must sleep, keep the duration short (1-5 seconds)`
}
```

## 设计要点

1. **动态检测**: 运行时检测PowerShell版本
2. **版本适配**: 提供版本特定的语法指导
3. **环境感知**: 根据环境变量调整内容

## 与其他文件的关系

- **PowerShellTool.tsx**: 调用getPrompt获取提示
- **utils/shell/powershellDetection.ts**: 版本检测
