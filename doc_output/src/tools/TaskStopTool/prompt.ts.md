# prompt.ts — 任务停止工具提示信息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskStopTool/prompt.ts`
- **类型**: TypeScript 提示信息模块
- **功能**: 定义 TaskStop 工具的名称常量

## 核心内容详解

### 导出常量

```typescript
export const TASK_STOP_TOOL_NAME = 'TaskStop'
```

此文件仅导出工具名称常量。工具的主要描述定义在 `constants.ts` 中。

## 与其他文件的关系

### 被依赖
- `TaskStopTool.ts` — 工具主实现文件
