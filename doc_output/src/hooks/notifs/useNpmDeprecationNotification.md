# useNpmDeprecationNotification.tsx — npm 弃用通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useNpmDeprecationNotification.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `useNpmDeprecationNotification`
- **依赖**: useStartupNotification, bundledMode, doctorDiagnostic, envUtils

## 功能概述

本 Hook 检测用户是否通过 npm 安装 Claude Code，如果是则显示弃用通知，引导用户使用原生安装器。

## 核心内容详解

### 弃用消息

```typescript
const NPM_DEPRECATION_MESSAGE = 
  'Claude Code has switched from npm to native installer. ' +
  'Run `claude install` or see https://docs.anthropic.com/en/docs/claude-code/getting-started for more options.'
```

### 检查逻辑

```typescript
async function check() {
  // 1. 捆绑模式或禁用检查则跳过
  if (isInBundledMode() || isEnvTruthy(process.env.DISABLE_INSTALLATION_CHECKS)) {
    return null
  }
  
  // 2. 获取安装类型
  const installationType = await getCurrentInstallationType()
  if (installationType === 'development') {
    return null
  }
  
  // 3. 返回通知
  return {
    timeoutMs: 15000,
    key: 'npm-deprecation-warning',
    text: NPM_DEPRECATION_MESSAGE,
    color: 'warning',
    priority: 'high',
  }
}
```

## 设计要点

### 1. 条件显示

- 不显示给捆绑模式用户（已经是原生安装）
- 不显示给开发环境（开发者知道自己在做什么）
- 可通过环境变量禁用

### 2. 异步检测

`getCurrentInstallationType()` 可能需要异步检查安装方式。

### 3. 长超时

15 秒超时，给用户足够时间阅读并采取行动。

### 4. 高优先级

npm 安装已不再支持，使用 'high' 优先级。

## 与其他文件的关系

- **useStartupNotification.ts**: 基础 Hook
- **bundledMode.ts**: 检测捆绑模式
- **doctorDiagnostic.ts**: 检测安装类型

## 注意事项

1. **向后兼容**: 仍然允许 npm 安装运行，但提示迁移
2. **环境变量**: `DISABLE_INSTALLATION_CHECKS` 可用于 CI 等场景禁用提示
3. **开发环境**: 开发时不显示，避免干扰
4. **安装类型**: 需要准确的安装类型检测
