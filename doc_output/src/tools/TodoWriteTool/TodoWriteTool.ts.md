# TodoWriteTool.ts — 待办写入工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TodoWriteTool/TodoWriteTool.ts`
- **类型**: TypeScript 工具模块
- **功能**: 管理当前会话的待办列表（V1 版本）

## 功能概述

TodoWriteTool 用于创建和管理结构化的待办列表。这是 V1 版本的任务管理工具，当 V2 版本（TodoV2）禁用时使用。

## 核心内容详解

### 输入 Schema

```typescript
{
  todos: TodoListSchema  // 更新后的待办列表
}
```

### 输出 Schema

```typescript
{
  oldTodos: TodoListSchema    // 更新前的待办列表
  newTodos: TodoListSchema    // 更新后的待办列表
  verificationNudgeNeeded?: boolean  // 是否需要验证提示
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `call()` | 执行待办列表更新 |
| `mapToolResultToToolResultBlockParam()` | 格式化结果为可读文本 |

### 核心逻辑

1. **获取现有待办**: 从 AppState 获取当前待办列表
2. **完成检测**: 检查所有任务是否已完成
3. **验证提示**: 如果完成 3+ 任务且无验证步骤，提示生成验证智能体
4. **状态更新**: 更新 AppState 中的待办列表

### 任务状态

- **pending**: 未开始
- **in_progress**: 进行中（同一时间只能有一个）
- **completed**: 已完成

## 设计要点

### 版本控制
- `isEnabled()` 返回 `!isTodoV2Enabled()` — V2 禁用时启用

### 结构验证提示
- 完成 3+ 任务且无验证步骤时提示生成验证智能体

### 权限检查
- 无需权限检查：`checkPermissions()` 始终返回 `allow`

## 与其他文件的关系

### 依赖
- `../../utils/todo/types.js` — 待办列表类型定义
- `../../utils/tasks.js` — 任务管理（用于 V2 检测）
- `../AgentTool/constants.js` — 验证智能体类型常量
- `./constants.ts` — 工具名称常量
- `./prompt.ts` — 工具提示信息

### 使用场景
- 跟踪复杂多步骤任务进度
- 管理会话待办清单
- V1 版本的任务管理

## 注意事项

1. **版本迁移**: 这是 V1 版本，V2 版本使用 TaskCreateTool 等工具
2. **严格模式**: 启用 `strict: true` 进行严格的输入验证
3. **自动清理**: 所有任务完成后自动清空列表
