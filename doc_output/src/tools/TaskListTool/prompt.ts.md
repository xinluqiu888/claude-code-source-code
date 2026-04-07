# prompt.ts — 任务列表工具提示信息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskListTool/prompt.ts`
- **类型**: TypeScript 提示信息模块
- **功能**: 定义 TaskList 工具的动态提示信息

## 核心内容详解

### 导出内容

```typescript
export const DESCRIPTION = 'List all tasks in the task list'

export function getPrompt(): string  // 动态生成
```

### 提示信息结构

#### 使用场景
- 查看可用任务（pending、无所有者、未被阻塞）
- 检查项目整体进度
- 查找被阻塞的任务
- 完成任务后检查新解锁的任务

#### 输出字段
- **id**: 任务标识符
- **subject**: 任务简要描述
- **status**: 任务状态
- **owner**: 智能体 ID（已分配时）
- **blockedBy**: 必须先解决的任务 ID 列表

#### 队友工作流（当启用时）
1. 完成任务后调用 TaskList 查找可用工作
2. 查找 pending 状态、无所有者、blockedBy 为空的任务
3. 优先按 ID 顺序选择（最低 ID 优先）
4. 使用 TaskUpdate 的 owner 参数认领任务
5. 被阻塞时通知团队负责人

## 设计要点

### 动态提示
- `getPrompt()` 函数支持根据功能开关动态生成内容
- `isAgentSwarmsEnabled()` 控制队友工作流相关提示

## 与其他文件的关系

### 被依赖
- `TaskListTool.ts` — 工具主实现文件
