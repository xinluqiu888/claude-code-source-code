# constants.ts — 任务停止工具常量

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskStopTool/constants.ts`
- **类型**: TypeScript 常量模块
- **功能**: 定义 TaskStop 工具的名称和描述常量

## 核心内容详解

### 导出常量

```typescript
export const TASK_STOP_TOOL_NAME = 'TaskStop'

export const DESCRIPTION = `
- Stops a running background task by its ID
- Takes a task_id parameter identifying the task to stop
- Returns a success or failure status
- Use this tool when you need to terminate a long-running task
`
```

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `TASK_STOP_TOOL_NAME` | `'TaskStop'` | 工具的唯一标识名称 |
| `DESCRIPTION` | 多行字符串 | 工具功能描述 |

## 与其他文件的关系

### 被依赖
- `TaskStopTool.ts` — 工具主实现文件
