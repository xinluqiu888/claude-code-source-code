# prompt.ts — 任务更新工具提示信息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskUpdateTool/prompt.ts`
- **类型**: TypeScript 提示信息模块
- **功能**: 定义 TaskUpdate 工具的提示信息

## 核心内容详解

### 导出内容

```typescript
export const DESCRIPTION = 'Update a task in the task list'

export const PROMPT = `...`
```

### 提示信息结构

#### 标记任务为已完成
- 完成工作时
- 任务不再需要或已被取代时
- **重要**: 总是标记分配的任务为已完成
- 完成后调用 TaskList 查找下一个任务

#### 完成要求
- 仅当完全完成时才标记为 completed
- 遇到错误或阻塞时保持 in_progress
- 被阻塞时创建新任务描述需要解决的内容
- 以下情况永远不要标记为 completed:
  - 测试失败
  - 实现不完整
  - 遇到未解决的错误
  - 找不到必要文件或依赖

#### 删除任务
- 任务不再相关或创建错误时
- 设置状态为 `deleted` 永久删除

#### 更新任务详情
- 需求变化或更清晰时
- 建立任务间依赖关系时

#### 可更新字段
- **status**: 任务状态
- **subject**: 任务标题（祈使形式）
- **description**: 任务描述
- **activeForm**: 进行时显示
- **owner**: 任务所有者
- **metadata**: 元数据
- **addBlocks**: 标记阻塞的任务
- **addBlockedBy**: 标记被阻塞的任务

#### 状态工作流
`pending` → `in_progress` → `completed`

使用 `deleted` 永久删除任务。

#### 过期检查
使用 `TaskGet` 读取任务最新状态后再更新。

## 与其他文件的关系

### 被依赖
- `TaskUpdateTool.ts` — 工具主实现文件
