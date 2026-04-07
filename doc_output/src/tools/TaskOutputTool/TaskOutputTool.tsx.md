# TaskOutputTool.tsx — 任务输出工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskOutputTool/TaskOutputTool.tsx`
- **类型**: TypeScript/React 工具模块
- **功能**: 获取后台任务的输出内容（已弃用）

## 功能概述

TaskOutputTool 用于从运行中或已完成的任务（后台 shell、智能体或远程会话）获取输出。此工具已被标记为弃用，推荐使用 Read 工具直接读取任务输出文件。

## 核心内容详解

### 输入 Schema

```typescript
{
  task_id: string   // 要获取输出的任务 ID
  block: boolean    // 是否等待完成（默认 true）
  timeout: number   // 最大等待时间（毫秒，默认 30000）
}
```

### 输出 Schema

```typescript
{
  retrieval_status: 'success' | 'timeout' | 'not_ready'
  task: TaskOutput | null
}

// TaskOutput 包含：
{
  task_id: string
  task_type: TaskType
  status: string
  description: string
  output: string
  exitCode?: number
  error?: string
  prompt?: string    // 智能体任务
  result?: string    // 智能体任务
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `getTaskOutputData()` | 获取任何任务类型的输出数据 |
| `waitForTaskCompletion()` | 等待任务完成（支持超时和中止） |
| `call()` | 执行输出获取操作 |
| `mapToolResultToToolResultBlockParam()` | 格式化结果为 XML 格式 |
| `renderToolResultMessage()` | React 渲染结果消息 |

### 支持的任务类型

- **local_bash**: 本地 Shell 任务
- **local_agent**: 本地智能体任务
- **remote_agent**: 远程智能体任务

## 设计要点

### 弃用状态
- 描述中明确标注 `[Deprecated]`
- 推荐使用 Read 工具直接读取任务输出文件路径

### 别名支持
- `aliases: ['AgentOutputTool', 'BashOutputTool']` — 向后兼容

### 阻塞与非阻塞模式
- `block: true`: 等待任务完成
- `block: false`: 立即返回当前状态

## 与其他文件的关系

### 依赖
- `../../utils/task/diskOutput.js` — 任务磁盘输出工具
- `../../utils/task/framework.js` — 任务框架工具
- `../../utils/task/outputFormatting.js` — 输出格式化工具
- `../AgentTool/UI.js` — 智能体 UI 组件
- `./constants.ts` — 工具名称常量

### 使用场景
- 获取后台任务输出（已弃用）
- 向后兼容旧版 SDK
