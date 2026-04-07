# prompt.ts — 任务获取工具提示信息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskGetTool/prompt.ts`
- **类型**: TypeScript 提示信息模块
- **功能**: 定义 TaskGet 工具的描述和提示信息

## 核心内容详解

### 导出内容

```typescript
export const DESCRIPTION = 'Get a task by ID from the task list'

export const PROMPT = `...`
```

### 提示信息结构

#### 使用场景
- 开始工作前获取完整描述和上下文
- 理解任务依赖关系（阻塞什么、被什么阻塞）
- 被分配任务后获取完整需求

#### 输出字段
- **subject**: 任务标题
- **description**: 详细需求和上下文
- **status**: 任务状态（pending, in_progress, completed）
- **blocks**: 等待此任务完成的任务
- **blockedBy**: 必须先完成的任务

#### 使用建议
- 获取任务后，验证 blockedBy 列表为空再开始工作
- 使用 TaskList 查看所有任务的摘要

## 与其他文件的关系

### 被依赖
- `TaskGetTool.ts` — 工具主实现文件
