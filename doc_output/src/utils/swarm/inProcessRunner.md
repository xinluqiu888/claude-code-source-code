# inProcessRunner.ts — 进程内队友运行器

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/inProcessRunner.ts`  
**关联模块**: Agent 执行、任务系统  
**主要依赖**: `src/tools/AgentTool/runAgent.js`, `src/utils/teammateContext.js`

## 功能概述

本文件包装 `runAgent()` 用于进程内 teammates，提供：
- 基于 `AsyncLocalStorage` 的上下文隔离
- 进度跟踪和 AppState 更新
- 完成时向领导发送空闲通知
- 计划模式审批流程支持
- 完成或中止时的清理

## 核心内容详解

### 主要功能

#### `startInProcessTeammate(params)`
启动进程内 teammate 的 agent 执行循环：
1. 使用 `runWithTeammateContext()` 执行 agent 循环
2. 附加 teammate 系统提示追加内容
3. 配置工具权限（允许的工具列表）
4. 设置进度跟踪
5. 处理空闲通知和清理

参数包含：
```typescript
{
  identity: TeammateIdentity       // 身份信息
  taskId: string                   // 任务 ID
  prompt: string                   // 初始提示
  teammateContext: TeammateContext // AsyncLocalStorage 上下文
  toolUseContext: ToolUseContext   // 工具使用上下文
  abortController: AbortController // 中止控制器
  model?: string                   // 模型覆盖
  systemPrompt?: string            // 自定义系统提示
  systemPromptMode?: string        // 系统提示模式
  allowedTools?: string[]          // 允许的工具列表
  allowPermissionPrompts?: boolean // 是否允许权限提示
}
```

#### `createInProcessCanUseTool(identity, abortController, onPermissionWaitMs?)`
创建进程内 teammate 的权限检查函数：
- 解析 'ask' 权限（通过 UI 而非拒绝）
- 优先使用领导的 ToolUseConfirm 对话框
- 回退到邮箱系统
- 支持分类器自动批准（仅 Bash 工具）

### 权限处理流程

1. **标准路径**: 使用 `ToolUseConfirm` 对话框显示 worker 徽章
2. **回退路径**: 使用邮箱系统发送请求到领导邮箱
3. **分类器**: Bash 命令尝试分类器自动批准
4. **回调注册**: 注册权限回调等待领导响应

### 状态管理

- **运行中**: Task status 为 'running'
- **等待计划批准**: `awaitingPlanApproval` 为 true
- **空闲**: `isIdle` 标记
- **已请求关闭**: `shutdownRequested` 标记
- **已中止**: `abortController.signal.aborted`

## 设计要点

1. **上下文隔离**: 使用 `runWithTeammateContext()` 确保每个 teammate 有自己的身份
2. **进度跟踪**: 通过 `createProgressTracker` 和 `appendTeammateMessage` 更新 UI
3. **空闲通知**: 完成时向领导发送 `createIdleNotification`
4. **权限桥接**: 通过 `leaderPermissionBridge` 访问领导的权限队列

## 与其他文件的关系

- **runAgent.ts**: 核心 agent 执行逻辑
- **teammateContext.ts**: 提供 `runWithTeammateContext()`
- **leaderPermissionBridge.ts**: 获取领导的权限队列 setter
- **InProcessTeammateTask.ts**: 调用 `startInProcessTeammate`

## 注意事项

- 进程内 teammates 与领导共享 API 客户端和 MCP 连接
- `permissionMode` 默认为 `plan`（如果 `planModeRequired`）或 `default`
- 等待权限的时间从显示时间中减去
- 工具结果内容可能被替换以节省内存
