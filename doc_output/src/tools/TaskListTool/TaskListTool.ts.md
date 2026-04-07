# TaskListTool.ts — 任务列表工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-source-code/src/tools/TaskListTool/TaskListTool.ts`
- **类型**: TypeScript 工具模块
- **功能**: 列出任务列表中的所有任务

## 功能概述

TaskListTool 是一个只读工具，用于获取所有任务的摘要信息，帮助用户：
- 查看可用任务（pending 状态、无所有者、未被阻塞）
- 检查项目整体进度
- 查找被阻塞的任务

## 核心内容详解

### 输入 Schema

```typescript
{}  // 无需输入参数
```

### 输出 Schema

```typescript
{
  tasks: Array<{
    id: string
    subject: string
    status: 'pending' | 'in_progress' | 'completed'
    owner?: string        // 分配给智能体时显示
    blockedBy: string[]   // 阻塞此任务的任务 ID
  }>
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `call()` | 获取过滤后的任务列表 |
| `mapToolResultToToolResultBlockParam()` | 格式化结果为可读列表 |

### 核心逻辑

1. **过滤内部任务**: 排除标记为 `_internal` 的任务
2. **构建已解决集合**: 已完成的任务不会显示为阻塞
3. **过滤阻塞列表**: 只显示未解决的阻塞任务

## 设计要点

### 只读特性
- `isReadOnly(): true` — 不会修改任何状态
- `isConcurrencySafe(): true` — 支持并发读取

### 功能开关
- 通过 `isTodoV2Enabled()` 控制是否启用

### 智能过滤
- 自动过滤已解决任务的阻塞关系
- 隐藏内部任务（_internal 标记）

## 与其他文件的关系

### 依赖
- `../../utils/tasks.js` — 任务管理工具函数
- `./constants.ts` — 工具名称常量
- `./prompt.ts` — 工具提示信息

### 使用场景
- 查找可用工作
- 检查项目进度
- 队友工作流中的任务发现
