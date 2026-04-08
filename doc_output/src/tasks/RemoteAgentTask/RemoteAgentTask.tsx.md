# RemoteAgentTask.tsx — 远程代理任务管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/RemoteAgentTask/RemoteAgentTask.tsx`
- **类型**: TSX 模块
- **主要功能**: 管理远程代理会话的生命周期，包括注册、轮询、完成检查和通知

## 功能概述

本文件实现了RemoteAgentTask，用于管理在云环境中运行的远程代理会话。支持多种远程任务类型(remote-agent, ultraplan, ultrareview等)，提供会话恢复、进度轮询、完成检测和通知功能。

## 核心内容详解

### 类型定义

#### `RemoteAgentTaskState`
远程代理任务状态：
```typescript
type RemoteAgentTaskState = TaskStateBase & {
  type: 'remote_agent'
  remoteTaskType: RemoteTaskType
  remoteTaskMetadata?: RemoteTaskMetadata  // PR号、仓库等
  sessionId: string                        // 原始会话ID
  command: string
  title: string
  todoList: TodoList
  log: SDKMessage[]
  isLongRunning?: boolean                  // 长运行代理
  pollStartedAt: number                    // 本地轮询开始时间
  isRemoteReview?: boolean                 // 是否远程审查
  reviewProgress?: {                       // 审查进度
    stage?: 'finding' | 'verifying' | 'synthesizing'
    bugsFound: number
    bugsVerified: number
    bugsRefuted: number
  }
  isUltraplan?: boolean
  ultraplanPhase?: Exclude<UltraplanPhase, 'running'>
}
```

#### `RemoteTaskType`
远程任务类型：
```typescript
type RemoteTaskType = 'remote-agent' | 'ultraplan' | 'ultrareview' | 'autofix-pr' | 'background-pr'
```

#### `RemoteTaskMetadata`
远程任务元数据：
```typescript
type AutofixPrRemoteTaskMetadata = {
  owner: string
  repo: string
  prNumber: number
}
```

### 主要函数

#### 完成检查器注册

##### `registerCompletionChecker`
注册远程任务类型的完成检查器：
- 在每个轮询tick调用
- 返回非null字符串表示任务完成
- 通过--resume持久化

#### 元数据持久化

##### `persistRemoteAgentMetadata`
持久化远程代理元数据到会话sidecar。

##### `removeRemoteAgentMetadata`
从会话sidecar移除远程代理元数据。

#### 资格检查

##### `checkRemoteAgentEligibility`
检查创建远程代理会话的资格：
- 检查背景远程会话前提条件
- 返回eligible或错误列表

##### `formatPreconditionError`
格式化前提错误为人类可读消息：
- not_logged_in: 需要登录
- no_remote_environment: 无云环境
- not_in_git_repo: 需要git仓库
- no_git_remote: 需要GitHub远程
- github_app_not_installed: 需要安装Claude GitHub应用
- policy_blocked: 组织策略阻止

#### 通知函数

##### `enqueueRemoteNotification`
发送远程任务完成/失败/终止通知。

##### `markTaskNotified`
原子性标记任务为已通知。

##### `enqueueUltraplanFailureNotification`
发送Ultraplan特定失败通知。

#### 内容提取

##### `extractPlanFromLog`
从远程会话日志提取计划内容(搜索`<ultraplan>`标签)。

##### `extractReviewFromLog`
从远程会话日志提取审查内容：
- 优先扫描hook_progress/hook_response(bughunter模式)
- 备选扫描assistant消息(prompt模式)
- 支持hook stdout拼接回退

##### `extractReviewTagFromLog`
标签专用变体，仅返回显式`<remote-review>`标签内容。

##### `extractTodoListFromLog`
从SDK消息提取待办列表(查找最后的TodoWrite工具使用)。

#### 任务注册与恢复

##### `registerRemoteAgentTask`
注册远程代理任务：
1. 生成任务ID
2. 创建输出文件
3. 创建任务状态
4. 注册到AppState
5. 持久化元数据到sidecar
6. 启动轮询

##### `restoreRemoteAgentTasks`
从会话sidecar恢复远程代理任务(--resume时)：
1. 扫描remote-agents/目录
2. 获取每个会话的实时CCR状态
3. 重建RemoteAgentTaskState
4. 为运行中会话重启轮询
5. 清理已归档/404的会话

#### 轮询逻辑

##### `startRemoteSessionPolling`
启动远程会话轮询：
- 轮询间隔：1秒
- 远程审查超时：30分钟
- 稳定空闲检测：5次连续空闲轮询

轮询处理流程：
1. 获取任务状态
2. 调用pollRemoteSessionEvents获取新事件
3. 追加事件到accumulatedLog
4. 检查会话状态(archived = 完成)
5. 运行完成检查器
6. 处理远程审查特定逻辑：
   - 扫描`<remote-review>`标签
   - 解析进度心跳
   - 检测稳定空闲
   - 超时检查
7. 更新任务状态
8. 发送完成通知

审查进度解析：
- 从hook_progress/hook_response解析`<remote-review-progress>`标签
- 提取stage、bugs_found、bugs_verified、bugs_refuted

完成条件：
- result成功/失败(非ultraplan/long-running)
- cachedReviewContent非null(审查模式)
- stableIdle且hasAssistantEvents且非bughunter模式
- 超时(30分钟)

### RemoteAgentTask 对象

实现Task接口：
```typescript
export const RemoteAgentTask: Task = {
  name: 'RemoteAgentTask',
  type: 'remote_agent',
  async kill(taskId, setAppState) {
    // 标记为killed，停止轮询
    // 远程会话保持运行，仅本地停止跟踪
  }
}
```

## 设计要点

1. **会话保持**：kill仅停止本地跟踪，远程会话保持运行供用户回访
2. **稳定空闲检测**：需要5次连续空闲轮询避免误判短暂空闲
3. **双模式审查**：支持bughunter(SessionStart hook)和prompt模式
4. **增量扫描**：只扫描新事件(delta)保持O(new)复杂度
5. **sidecar持久化**：支持--resume恢复远程任务
6. **标签解析**：支持`<remote-review>`, `<ultraplan>`, `<remote-review-progress>`标签

## 与其他文件的关系

- **导入**：
  - SDK类型、XML常量、产品URL
  - Task接口、TodoWriteTool
  - 远程会话API、元数据存储
  - 各种工具函数

- **被调用者**：
  - AgentTool - 创建远程代理
  - resume流程 - 恢复远程任务

## 注意事项

1. **远程审查超时**：30分钟硬限制，防止无限挂起
2. **标签扫描**：新事件delta扫描，避免重复处理完整日志
3. **hook事件计数**：bughunter模式下hook_progress/hook_response计入输出
4. **404处理**：仅404视为真正丢失，401等可恢复错误会重试
5. **Ultraplan例外**：Ultraplan和long-running任务跳过result驱动完成
