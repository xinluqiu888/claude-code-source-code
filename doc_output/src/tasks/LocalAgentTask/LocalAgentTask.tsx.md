# LocalAgentTask.tsx — 后台代理执行管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/LocalAgentTask/LocalAgentTask.tsx`
- **类型**: TSX 模块
- **主要功能**: 处理后台代理任务的执行、进度跟踪和生命周期管理

## 功能概述

本文件实现了LocalAgentTask，用于管理后台代理执行。替代了原有的AsyncAgent实现，提供统一的Task接口。支持代理进度跟踪、token计数、工具活动记录，以及前台/后台状态切换。

## 核心内容详解

### 类型定义

#### `ToolActivity`
工具活动记录：
```typescript
type ToolActivity = {
  toolName: string
  input: Record<string, unknown>
  activityDescription?: string  // 预计算的活动描述
  isSearch?: boolean            // 是否为搜索操作
  isRead?: boolean              // 是否为读取操作
}
```

#### `AgentProgress`
代理进度信息：
```typescript
type AgentProgress = {
  toolUseCount: number
  tokenCount: number
  lastActivity?: ToolActivity
  recentActivities?: ToolActivity[]
  summary?: string              // 后台摘要
}
```

#### `ProgressTracker`
进度追踪器(运行时使用)：
```typescript
type ProgressTracker = {
  toolUseCount: number
  latestInputTokens: number        // 最新输入token(累积)
  cumulativeOutputTokens: number   // 累积输出token
  recentActivities: ToolActivity[]
}
```

#### `LocalAgentTaskState`
本地代理任务状态：
- `type`: 'local_agent'
- `agentId`: string - 代理ID
- `prompt`: string - 提示
- `selectedAgent?`: AgentDefinition - 选中的代理定义
- `agentType`: string - 代理类型
- `model?`: string - 模型覆盖
- `abortController?`: AbortController - 终止控制器
- `progress?`: AgentProgress - 进度信息
- `isBackgrounded`: boolean - 是否后台运行
- `pendingMessages`: string[] - 待处理消息队列
- `retain`: boolean - UI是否持有此任务
- `diskLoaded`: boolean - 是否已从磁盘加载
- `evictAfter?`: number - 驱逐时间戳

### 主要函数

#### 进度跟踪

##### `createProgressTracker`
创建新的进度追踪器实例。

##### `updateProgressFromMessage`
从消息更新进度：
- 处理输入token(取最新值，API中是累积的)
- 累加输出token
- 记录工具使用活动(排除StructuredOutput)
- 限制recentActivities最多5条

##### `getProgressUpdate`
从追踪器获取进度更新。

##### `createActivityDescriptionResolver`
从工具列表创建活动描述解析器。

#### 消息队列管理

##### `queuePendingMessage`
向任务待处理消息队列添加消息。

##### `appendMessageToLocalAgent`
追加消息到task.messages(立即显示在转录中)。

##### `drainPendingMessages`
清空并返回所有待处理消息。

#### 任务生命周期

##### `LocalAgentTask.kill`
终止代理任务：
- 调用abortController.abort()
- 清理unregisterCleanup
- 设置状态为'killed'
- 安排任务输出驱逐

##### `killAsyncAgent`
终止异步代理(同上)。

##### `killAllRunningAgentTasks`
终止所有运行中的代理任务。

##### `markAgentsNotified`
批量标记代理为已通知(用于chat:killAgents)。

#### 进度更新

##### `updateAgentProgress`
更新代理进度，保留现有summary字段。

##### `updateAgentSummary`
更新代理后台摘要：
- 存储1-2句进度摘要
- 向SDK消费者发送进度事件(如果启用)

#### 任务完成

##### `completeAgentTask`
完成任务并设置结果：
- 设置状态为'completed'
- 存储AgentToolResult
- 清理资源

##### `failAgentTask`
标记任务失败：
- 设置状态为'failed'
- 存储错误信息
- 清理资源

#### 任务注册

##### `registerAsyncAgent`
注册异步代理任务：
- 初始化任务输出符号链接
- 创建abortController(支持父子关系)
- 设置任务状态
- 注册清理处理器

##### `registerAgentForeground`
注册前台代理任务(可后续转为后台)：
- 创建backgroundSignal Promise
- 支持autoBackgroundMs自动后台化
- 返回cancelAutoBackground函数

##### `backgroundAgentTask`
将前台代理转为后台：
- 设置isBackgrounded为true
- 解析backgroundSignal

## 设计要点

1. **Token计数策略**: 输入token取最新(累积)，输出token累加
2. **活动分类**: 预计算isSearch/isRead标志用于UI展示
3. **前台/后台切换**: 支持代理从 foreground 转为 background 运行
4. **消息队列**: pendingMessages支持通过SendMessage在回合间发送消息
5. **驱逐策略**: evictAfter控制任务面板的清理时机
6. **父子AbortController**: 支持子代理随父代理自动终止

## 与其他文件的关系

- **导入**:
  - SDK进度摘要配置、XML常量
  - AppState, Task接口
  - AgentToolResult, AgentDefinition
  - 各种工具函数(中止控制器、清理注册、任务框架等)

- **被调用者**:
  - AgentTool - 创建和管理代理任务
  - 协调器模式组件 - 管理子代理

## 注意事项

1. **进度摘要门控**: SDK进度事件仅在getSdkAgentProgressSummariesEnabled()返回true时发送
2. **StructuredOutput排除**: 内部工具不计入活动预览
3. **保留任务**: retain标志阻止驱逐并启用流追加
4. **面板代理过滤**: isPanelAgentTask排除main-session任务
