# prompt.ts — 任务创建工具提示信息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskCreateTool/prompt.ts`
- **类型**: TypeScript 提示信息模块
- **功能**: 定义 TaskCreate 工具的描述和提示信息

## 核心内容详解

### 导出内容

```typescript
export const DESCRIPTION = 'Create a new task in the task list'

export function getPrompt(): string
```

### 提示信息结构

`getPrompt()` 返回的提示信息包含：

#### 使用场景
- 复杂多步骤任务（3个或以上步骤）
- 非平凡复杂任务
- 计划模式
- 用户明确要求待办列表
- 用户提供多个任务列表
- 接收新指令后
- 开始工作时
- 完成任务后

#### 不使用场景
- 单一简单任务
- 微不足道的任务
- 少于3个简单步骤的任务
- 纯对话或信息性任务

#### 任务字段
- **subject**: 祈使形式的简短标题
- **description**: 需要完成的内容
- **activeForm**: 可选，进行时在加载器中显示

## 设计要点

### 动态提示
- `getPrompt()` 是函数而非常量，支持动态生成
- 根据 `isAgentSwarmsEnabled()` 添加队友相关的使用建议

### 多智能体支持
当启用智能体群时，额外提示：
- 包含足够详细描述供其他智能体理解
- 新任务创建为 `pending` 状态，无所有者
- 使用 `TaskUpdate` 的 `owner` 参数分配任务

## 与其他文件的关系

### 被依赖
- `TaskCreateTool.ts` — 工具主实现文件
