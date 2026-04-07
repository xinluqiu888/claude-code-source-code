# interactiveHandler.ts — 交互式权限处理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/toolPermission/handlers/interactiveHandler.ts`
- **类型**: TypeScript 模块
- **导出函数**: `handleInteractivePermission`
- **导出类型**: `InteractivePermissionParams`

## 功能概述

本模块处理主代理（Main Agent）的交互式权限流程，支持：
1. 将权限请求推送到确认队列显示给用户
2. 并行处理多种审批来源（用户交互、钩子、分类器、CCR Bridge、MCP 通道）
3. 竞态模式：第一个响应者获胜
4. 支持取消和中止操作
5. 分类器自动审批的可视化反馈

## 核心内容详解

### 参数类型

```typescript
type InteractivePermissionParams = {
  ctx: PermissionContext           // 权限上下文
  description: string              // 工具描述
  result: PermissionDecision & { behavior: 'ask' }  // 初始权限结果
  awaitAutomatedChecksBeforeDialog: boolean | undefined  // 是否在显示对话框前等待自动化检查
  bridgeCallbacks?: BridgePermissionCallbacks  // CCR Bridge 回调
  channelCallbacks?: ChannelPermissionCallbacks  // MCP 通道回调
}
```

### 核心机制：竞态处理

使用 `createResolveOnce` 确保只有一个响应者能最终解决权限决策：
```typescript
const { resolve: resolveOnce, isResolved, claim } = createResolveOnce(resolve)
```

所有回调都通过 `claim()` 进行原子性检查，避免竞态条件。

### 用户交互回调

**onUserInteraction()**
- 用户开始与权限对话框交互时调用
- 200ms 宽限期防止误触
- 清除分类器检查指示器

**onDismissCheckmark()**
- 用户按 Esc 取消分类器自动批准的对勾
- 清除定时器和监听器

**onAbort()**
- 用户中止权限请求
- 通知 CCR Bridge 和 MCP 通道
- 记录决策并解决为拒绝

**onAllow()**
- 用户允许权限请求
- 支持更新输入、权限更新、反馈、内容块
- 异步调用 `ctx.handleUserAllow`

**onReject()**
- 用户拒绝权限请求
- 支持反馈消息
- 记录决策并解决为拒绝

**recheckPermission()**
- 重新检查权限（如 CCR 端模式切换后）
- 如果权限已允许，自动解决

### CCR Bridge 集成

当 `bridgeCallbacks` 存在时：
1. 发送权限请求到 CCR（claude.ai）
2. 订阅响应回调
3. 支持 `allow` 和 `deny` 两种响应
4. 支持 `updatedInput` 和 `updatedPermissions`

### MCP 通道集成

当 `channelCallbacks` 存在且工具不需要用户交互时：
1. 向所有活跃通道（Telegram、iMessage 等）发送权限请求通知
2. 订阅通道响应
3. 响应格式为 "yes abc123" 或 "no abc123"
4. 在通知处理程序中拦截回复

### 钩子执行

如果 `awaitAutomatedChecksBeforeDialog` 为 false：
1. 异步执行权限请求钩子
2. 如果钩子返回决策，竞争获胜后解决

### 分类器自动审批

当满足以下条件时：
- `feature('BASH_CLASSIFIER')` 启用
- 存在 `pendingClassifierCheck`
- 工具为 BASH_TOOL
- `awaitAutomatedChecksBeforeDialog` 为 false

执行流程：
1. 设置分类器检查指示器
2. 异步执行分类器检查
3. 批准时显示对勾动画（3秒聚焦，1秒非聚焦）
4. 支持提前通过 Esc 取消

## 设计要点

### 1. 多源竞态

本地用户、CCR Bridge、MCP 通道、钩子、分类器五路并行，先到先得。

### 2. 取消传播

权限取消需要通知所有远程端（Bridge、通道）避免状态不一致。

### 3. 优雅降级

CCR Bridge 和 MCP 通道失败时不影响本地对话框。

### 4. 结构化参数

MCP 通道使用结构化参数而非文本猜测，服务器负责消息格式化。

### 5. 分类器反馈

分类器自动批准后显示视觉反馈（对勾），让用户感知自动决策。

## 与其他文件的关系

- **PermissionContext.ts**: 使用 `PermissionContext` 和 `createResolveOnce`
- **bashPermissions.ts**: 调用 `executeAsyncClassifierCheck`
- **bridgePermissionCallbacks.ts**: 使用 `BridgePermissionCallbacks`
- **channelPermissions.ts**: 使用 `ChannelPermissionCallbacks`
- **classifierApprovals.ts**: 调用 `setClassifierChecking`, `clearClassifierChecking`

## 注意事项

1. **回调注册顺序**: MCP 回调必须在发送请求前注册，避免竞态
2. **AbortSignal 清理**: 所有信号监听器都需要在解决后清理
3. **200ms 宽限期**: 防止用户无意按键取消分类器
4. **泛型工具支持**: CCR 支持任何工具的通用允许/拒绝，但 `updatedInput` 只在有专用渲染器时返回
5. **通道纯 yes/no**: 通道回复不支持 `updatedInput`，仅限纯批准/拒绝
