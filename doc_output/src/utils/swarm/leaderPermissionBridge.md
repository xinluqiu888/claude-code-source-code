# leaderPermissionBridge.ts — 领导权限桥接

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/leaderPermissionBridge.ts`  
**关联模块**: 权限系统、REPL  
**主要依赖**: `src/components/permissions/PermissionRequest.js`, `src/Tool.js`

## 功能概述

本文件提供模块级桥接，允许 REPL 注册其 `setToolUseConfirmQueue` 和 `setToolPermissionContext` 函数供进程内 teammates 使用。

当进程内 teammate 请求权限时，使用标准的 ToolUseConfirm 对话框而非 worker 权限徽章。此桥接使 REPL 的队列 setter 和权限上下文 setter 可从进程内运行器访问。

## 核心内容详解

### 类型定义

```typescript
type SetToolUseConfirmQueueFn = (
  updater: (prev: ToolUseConfirm[]) => ToolUseConfirm[]
) => void

type SetToolPermissionContextFn = (
  context: ToolPermissionContext,
  options?: { preserveMode?: boolean }
) => void
```

### 核心函数

#### `registerLeaderToolUseConfirmQueue(setter)`
注册领导的 `setToolUseConfirmQueue` 函数。

#### `getLeaderToolUseConfirmQueue()`
获取已注册的队列 setter，如果不存在返回 `null`。

#### `unregisterLeaderToolUseConfirmQueue()`
取消注册队列 setter（设为 `null`）。

#### `registerLeaderSetToolPermissionContext(setter)`
注册领导的 `setToolPermissionContext` 函数。

#### `getLeaderSetToolPermissionContext()`
获取已注册的权限上下文 setter，如果不存在返回 `null`。

#### `unregisterLeaderSetToolPermissionContext()`
取消注册权限上下文 setter。

## 设计要点

1. **模块级状态**: 使用模块级变量存储注册的 setter，避免 props 传递
2. **可选依赖**: 返回 `null` 而非抛出错误，支持优雅降级
3. **生命周期管理**: 提供注册和取消注册函数，支持热重载

## 与其他文件的关系

- **inProcessRunner.ts**: 调用 `getLeaderToolUseConfirmQueue()` 和 `getLeaderSetToolPermissionContext()`
- **PermissionRequest.js**: 提供 `ToolUseConfirm` 类型
- **Tool.js**: 提供 `ToolPermissionContext` 类型

## 注意事项

- 必须在 REPL 初始化时注册，否则进程内 teammates 无法显示权限对话框
- 未注册时回退到邮箱系统
- `preserveMode` 选项防止 worker 的 `acceptEdits` 上下文泄漏回协调器
