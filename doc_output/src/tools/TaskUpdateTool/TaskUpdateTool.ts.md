# TaskUpdateTool.ts — 任务更新工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskUpdateTool/TaskUpdateTool.ts`
- **类型**: TypeScript 工具模块
- **功能**: 更新任务列表中的任务

## 功能概述

TaskUpdateTool 用于更新任务的各种属性，包括：
- 更新任务状态（pending → in_progress → completed）
- 修改任务标题和描述
- 分配任务所有者
- 设置任务依赖关系
- 删除任务

## 核心内容详解

### 输入 Schema

```typescript
{
  taskId: string
  subject?: string           // 新标题
  description?: string       // 新描述
  activeForm?: string        // 新进行时
  status?: 'pending' | 'in_progress' | 'completed' | 'deleted'
  owner?: string             // 新所有者
  addBlocks?: string[]       // 添加阻塞的任务 ID
  addBlockedBy?: string[]    // 添加被阻塞的任务 ID
  metadata?: Record<string, unknown>  // 元数据
}
```

### 输出 Schema

```typescript
{
  success: boolean
  taskId: string
  updatedFields: string[]
  error?: string
  statusChange?: { from: string; to: string }
  verificationNudgeNeeded?: boolean  // 内部使用
}
```

### 主要功能

#### 状态更新
- `pending` → `in_progress` → `completed` 标准流程
- `deleted`: 永久删除任务
- 标记 completed 时执行 TaskCompleted 钩子

#### 自动分配所有者
- 队友模式下，标记 `in_progress` 时自动设置所有者

#### 通知机制
- 所有权变更时通过 mailbox 通知新所有者

#### 依赖管理
- `addBlocks`: 标记此任务阻塞的其他任务
- `addBlockedBy`: 标记阻塞此任务的其他任务

## 设计要点

### 验证机制
- 完成任务时执行钩子验证
- 钩子返回阻塞错误时阻止状态变更

### 多智能体支持
- `isAgentSwarmsEnabled()` 控制队友功能
- 自动分配所有者
- 通过 mailbox 发送任务分配通知

### 结构验证提示
- 完成 3+ 任务且无验证步骤时，提示生成验证智能体

## 与其他文件的关系

### 依赖
- `../../utils/tasks.js` — 任务管理工具函数
- `../../utils/teammate.js` — 队友信息获取
- `../../utils/teammateMailbox.js` — 邮箱通知
- `../../utils/hooks.js` — 任务完成钩子
- `./constants.ts` — 工具名称常量
- `./prompt.ts` — 工具提示信息

### 使用场景
- 标记任务完成
- 更新任务进度
- 分配任务给队友
