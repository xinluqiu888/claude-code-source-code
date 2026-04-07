# TaskGetTool.ts — 任务获取工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskGetTool/TaskGetTool.ts`
- **类型**: TypeScript 工具模块
- **功能**: 通过 ID 从任务列表中获取任务详情

## 功能概述

TaskGetTool 是一个只读工具，用于根据任务 ID 检索任务的完整信息，包括：
- 任务主题和描述
- 任务状态（pending, in_progress, completed）
- 任务依赖关系（blocks 和 blockedBy）

## 核心内容详解

### 输入 Schema

```typescript
{
  taskId: string  // 要检索的任务 ID
}
```

### 输出 Schema

```typescript
{
  task: {
    id: string
    subject: string
    description: string
    status: 'pending' | 'in_progress' | 'completed'
    blocks: string[]      // 此任务阻塞的任务 ID 列表
    blockedBy: string[]   // 阻塞此任务的任务 ID 列表
  } | null
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `call()` | 获取任务详情，返回任务对象或 null |
| `mapToolResultToToolResultBlockParam()` | 格式化结果为可读文本 |

## 设计要点

### 只读特性
- `isReadOnly(): true` — 不会修改任何状态
- `isConcurrencySafe(): true` — 支持并发读取

### 功能开关
- 通过 `isTodoV2Enabled()` 控制是否启用

### 错误处理
- 任务不存在时返回 `{ task: null }` 而非抛出错误

## 与其他文件的关系

### 依赖
- `../../utils/tasks.js` — 任务管理工具函数
- `./constants.ts` — 工具名称常量
- `./prompt.ts` — 工具提示信息

### 使用场景
- 开始工作前获取任务详情
- 检查任务依赖关系
- 被分配任务后获取完整需求
