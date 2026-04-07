# constants.ts — 任务创建工具常量

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskCreateTool/constants.ts`
- **类型**: TypeScript 常量模块
- **功能**: 定义 TaskCreate 工具的名称常量

## 核心内容详解

### 导出常量

```typescript
export const TASK_CREATE_TOOL_NAME = 'TaskCreate'
```

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `TASK_CREATE_TOOL_NAME` | `'TaskCreate'` | 工具的唯一标识名称 |

## 设计要点

- 采用集中式常量管理，避免硬编码字符串
- 工具名称用于工具注册、调用和识别

## 与其他文件的关系

### 被依赖
- `TaskCreateTool.ts` — 工具主实现文件
- 其他需要引用工具名称的文件
