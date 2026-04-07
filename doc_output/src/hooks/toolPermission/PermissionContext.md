# PermissionContext.ts — 工具权限上下文管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/toolPermission/PermissionContext.ts`
- **类型**: TypeScript 模块
- **导出函数**: `createPermissionContext`, `createPermissionQueueOps`, `createResolveOnce`
- **导出类型**: `PermissionContext`, `PermissionApprovalSource`, `PermissionQueueOps`, `PermissionRejectionSource`, `ResolveOnce`

## 功能概述

本文件是 Claude Code 工具权限系统的核心上下文管理模块，负责：
1. 创建权限检查上下文，包含工具、输入、消息等关键信息
2. 提供权限决策日志记录
3. 支持权限持久化和更新
4. 集成 BASH 分类器自动审批
5. 执行权限请求钩子（PermissionRequest hooks）
6. 构建权限决策（允许/拒绝）

## 核心内容详解

### 权限决策源类型

**PermissionApprovalSource** - 批准来源
```typescript
| { type: 'hook'; permanent?: boolean }      // 钩子自动批准
| { type: 'user'; permanent: boolean }       // 用户手动批准
| { type: 'classifier' }                     // 分类器自动批准
```

**PermissionRejectionSource** - 拒绝来源
```typescript
| { type: 'hook' }                           // 钩子拒绝
| { type: 'user_abort' }                     // 用户中止
| { type: 'user_reject'; hasFeedback: boolean }  // 用户拒绝
```

### createPermissionContext 函数

创建权限上下文对象，包含以下方法：

**日志相关**
- `logDecision(args, opts)`: 记录权限决策（接受/拒绝）
- `logCancelled()`: 记录权限取消事件

**持久化**
- `persistPermissions(updates)`: 持久化权限更新到配置

**中止处理**
- `resolveIfAborted(resolve)`: 如果已中止则立即解决
- `cancelAndAbort(feedback, isAbort, contentBlocks)`: 取消并中止，返回拒绝决策

**分类器（BASH_CLASSIFIER 功能）**
- `tryClassifier(pendingClassifierCheck, updatedInput)`: 尝试分类器自动审批

**钩子执行**
- `runHooks(permissionMode, suggestions, updatedInput, permissionPromptStartTimeMs)`: 执行权限请求钩子

**决策构建**
- `buildAllow(updatedInput, opts)`: 构建允许决策
- `buildDeny(message, decisionReason)`: 构建拒绝决策

**用户操作处理**
- `handleUserAllow(...)`: 处理用户允许操作，持久化权限
- `handleHookAllow(...)`: 处理钩子允许操作

**队列操作**
- `pushToQueue(item)`: 将权限请求推入队列
- `removeFromQueue()`: 从队列移除
- `updateQueueItem(patch)`: 更新队列项

### createResolveOnce 工具函数

提供原子性的一次性解决机制，防止竞态条件：
```typescript
type ResolveOnce<T> = {
  resolve(value: T): void
  isResolved(): boolean
  claim(): boolean  // 原子检查并标记
}
```

### createPermissionQueueOps 函数

将 React 的 `setToolUseConfirmQueue` 转换为通用的队列操作接口：
- `push(item)`: 添加项目
- `remove(toolUseID)`: 移除指定项目
- `update(toolUseID, patch)`: 更新指定项目

## 设计要点

### 1. 函数式上下文

使用工厂函数创建上下文对象而非类，便于组合和测试。

### 2. 不可变性

上下文对象通过 `Object.freeze(ctx)` 冻结，防止外部修改。

### 3. 竞态安全

`createResolveOnce` 使用 `claim()` 模式确保异步回调的原子性：
```typescript
if (!claim()) return // 已被其他回调处理
// 安全执行后续操作
```

### 4. 分类器集成

仅在 `feature('BASH_CLASSIFIER')` 启用时添加分类器方法，通过展开运算符动态构建对象。

### 5. 输入比较

使用 `tool.inputsEquivalent` 进行输入比较，支持自定义比较逻辑。

## 与其他文件的关系

- **permissionLogging.ts**: 提供 `logPermissionDecision`
- **bashPermissions.ts**: 提供 `awaitClassifierAutoApproval`
- **PermissionUpdate.ts**: 提供 `persistPermissionUpdates`, `applyPermissionUpdates`
- **hooks.ts**: 提供 `executePermissionRequestHooks`
- **PermissionRequest.tsx**: 使用 `PermissionQueueOps` 类型

## 注意事项

1. **分类器特性标志**: BASH_CLASSIFIER 功能通过编译时特性标志控制
2. **AbortSignal**: 所有异步操作都应检查 abortController.signal
3. **子代理检测**: 通过 `!!toolUseContext.agentId` 判断是否为子代理
4. **反馈处理**: 用户反馈会去除首尾空白
5. **错误转换**: 使用 `toError` 统一错误格式
