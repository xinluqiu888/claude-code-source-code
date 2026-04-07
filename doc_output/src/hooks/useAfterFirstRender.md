# useAfterFirstRender.ts — 首屏渲染后自动退出

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/useAfterFirstRender.ts`
- **类型**: React Hook
- **导出函数**: `useAfterFirstRender`
- **依赖**: React useEffect, envUtils

## 功能概述

本 Hook 用于性能测试场景，在组件首次渲染后自动输出启动时间并退出进程。主要用于：
1. 测量应用冷启动时间
2. CI/CD 中的性能回归测试
3. 内部开发调试

## 核心内容详解

### 触发条件

需要同时满足以下两个条件才会执行退出逻辑：

1. `process.env.USER_TYPE === 'ant'` — 仅限 Anthropic 内部员工
2. `isEnvTruthy(process.env.CLAUDE_CODE_EXIT_AFTER_FIRST_RENDER)` — 环境变量设置为真值

### 执行流程

```typescript
useEffect(() => {
  if (
    process.env.USER_TYPE === 'ant' &&
    isEnvTruthy(process.env.CLAUDE_CODE_EXIT_AFTER_FIRST_RENDER)
  ) {
    process.stderr.write(
      `\nStartup time: ${Math.round(process.uptime() * 1000)}ms\n`,
    )
    process.exit(0)
  }
}, [])
```

1. 使用 `process.uptime()` 获取 Node.js 进程运行时间
2. 转换为毫秒并输出到 stderr
3. 调用 `process.exit(0)` 正常退出

## 设计要点

### 1. 安全限制

- 仅限内部员工使用（`USER_TYPE=ant`），避免影响普通用户
- 需要通过显式环境变量启用

### 2. 输出位置

- 输出到 stderr 而非 stdout，避免干扰正常输出流
- 格式化输出包含换行符，便于解析

### 3. 副作用处理

- 依赖数组为空 `[]`，只在组件挂载时执行一次
- 使用 `useEffect` 确保在渲染完成后执行

## 与其他文件的关系

- **envUtils.ts**: 提供 `isEnvTruthy()` 工具函数，解析环境变量布尔值
- **应用入口**: 在根组件中调用此 Hook

## 注意事项

1. **进程退出**: 使用 `process.exit(0)` 是强制性的，会立即终止整个应用
2. **ESLint 忽略**: 代码中明确禁用 `custom-rules/no-process-exit` 规则
3. **精度限制**: `process.uptime()` 返回秒级精度，乘以 1000 转换为毫秒
4. **非生产代码**: 此功能仅用于开发和测试，不应在生产环境中启用
