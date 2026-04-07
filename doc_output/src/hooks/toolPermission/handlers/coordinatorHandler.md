# coordinatorHandler.ts — 协调器权限处理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/toolPermission/handlers/coordinatorHandler.ts`
- **类型**: TypeScript 模块
- **导出函数**: `handleCoordinatorPermission`
- **导出类型**: `CoordinatorPermissionParams`

## 功能概述

本模块处理协调器工作器（Coordinator Worker）的权限流程：
1. 按顺序等待自动化检查（钩子和分类器）
2. 如果自动化检查解决权限，直接返回决策
3. 如果都未解决，返回 null 让调用者回退到交互式对话框

## 核心内容详解

### 参数类型

```typescript
type CoordinatorPermissionParams = {
  ctx: PermissionContext                    // 权限上下文
  pendingClassifierCheck?: PendingClassifierCheck | undefined  // 待处理的分类器检查
  updatedInput: Record<string, unknown> | undefined  // 更新的输入
  suggestions: PermissionUpdate[] | undefined  // 权限建议
  permissionMode: string | undefined         // 权限模式
}
```

### 处理流程

```
1. 尝试权限钩子（快速、本地）
   ↓ 如果解决
   返回钩子决策
   ↓ 未解决
2. 尝试分类器（慢速、推理 -- 仅 bash）
   ↓ 如果解决
   返回分类器决策
   ↓ 未解决 或 出错
3. 返回 null（调用者应回退到交互式对话框）
```

### 钩子执行

```typescript
const hookResult = await ctx.runHooks(
  permissionMode,
  suggestions,
  updatedInput,
)
if (hookResult) return hookResult
```

### 分类器执行

仅在 `feature('BASH_CLASSIFIER')` 启用时执行：
```typescript
const classifierResult = feature('BASH_CLASSIFIER')
  ? await ctx.tryClassifier?.(params.pendingClassifierCheck, updatedInput)
  : null
if (classifierResult) {
  return classifierResult
}
```

### 错误处理

如果自动化检查意外失败：
1. 记录错误日志
2. 回退到交互式对话框（返回 null）
3. 非 Error 抛出会添加上下文前缀

```typescript
try {
  // 钩子或分类器检查
} catch (error) {
  if (error instanceof Error) {
    logError(error)
  } else {
    logError(new Error(`Automated permission check failed: ${String(error)}`))
  }
}
```

## 设计要点

### 1. 顺序执行

与交互式处理器的并行竞态不同，协调器按顺序等待自动化检查。

### 2. 特性标志保护

分类器检查受 `feature('BASH_CLASSIFIER')` 保护，编译时可剔除。

### 3. 容错设计

自动化检查失败不中断流程，而是回退到交互式对话框。

### 4. 无竞态问题

由于是顺序执行，不需要 `createResolveOnce` 等竞态保护机制。

## 与其他文件的关系

- **PermissionContext.ts**: 使用 `PermissionContext`
- **permissions.ts**: 定义 `PendingClassifierCheck`
- **interactiveHandler.ts**: 当此处理器返回 null 时，调用者会使用交互式处理器

## 注意事项

1. **与交互式处理器的区别**: 协调器是顺序等待，交互式是并行竞态
2. **分类器可选**: `tryClassifier` 可能不存在（非 BASH 工具）
3. **返回值语义**: null 表示"未解决，需要交互式对话框"
4. **错误日志**: 仅记录 Error 实例，其他值添加前缀后记录
