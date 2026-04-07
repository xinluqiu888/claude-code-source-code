# prompt.ts — 待办写入工具提示信息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TodoWriteTool/prompt.ts`
- **类型**: TypeScript 提示信息模块
- **功能**: 定义 TodoWrite 工具的提示信息

## 核心内容详解

### 导出内容

```typescript
export const PROMPT = `...`

export const DESCRIPTION = 'Update the todo list...'
```

### 提示信息结构

#### 使用场景
1. 复杂多步骤任务（3个或以上步骤）
2. 非平凡复杂任务
3. 用户明确要求待办列表
4. 用户提供多个任务列表
5. 接收新指令后
6. 开始工作时（标记为 in_progress）
7. 完成任务后

#### 不使用场景
1. 单一简单任务
2. 微不足道的任务
3. 少于3个简单步骤的任务
4. 纯对话或信息性任务

#### 任务状态
- **pending**: 未开始
- **in_progress**: 进行中（同一时间只能有一个）
- **completed**: 已完成

**重要**: 任务描述需要两种形式：
- **content**: 祈使形式（如 "Run tests"）
- **activeForm**: 进行时（如 "Running tests"）

#### 任务管理
- 实时更新任务状态
- 立即标记完成（不要批量完成）
- 同一时间只能有一个 in_progress
- 先完成当前任务再开始新任务
- 删除不再相关的任务

#### 任务完成要求
- 仅当完全完成时才标记为 completed
- 遇到错误或阻塞时保持 in_progress
- 被阻塞时创建新任务描述需要解决的内容
- 以下情况永远不要标记为 completed:
  - 测试失败
  - 实现不完整
  - 遇到未解决的错误
  - 找不到必要文件或依赖

#### 任务分解
- 创建具体、可操作的项目
- 将复杂任务分解为更小的可管理步骤
- 使用清晰、描述性的任务名称

## 与其他文件的关系

### 依赖
- `../FileEditTool/constants.js` — FileEditTool 名称常量

### 被依赖
- `TodoWriteTool.ts` — 工具主实现文件
