# TaskStopTool.ts — 任务停止工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskStopTool/TaskStopTool.ts`
- **类型**: TypeScript 工具模块
- **功能**: 通过 ID 停止正在运行的后台任务

## 功能概述

TaskStopTool 用于终止正在运行的后台任务。支持通过 task_id 或 shell_id（向后兼容）参数停止任务。

## 核心内容详解

### 输入 Schema

```typescript
{
  task_id?: string  // 要停止的任务 ID（优先）
  shell_id?: string  // 向后兼容的 KillShell 参数
}
```

### 输出 Schema

```typescript
{
  message: string   // 操作状态消息
  task_id: string   // 已停止的任务 ID
  task_type: string // 已停止的任务类型
  command?: string  // 已停止任务的命令或描述
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `validateInput()` | 验证任务存在且正在运行 |
| `call()` | 执行停止任务操作 |
| `mapToolResultToToolResultBlockParam()` | 格式化结果为 JSON |

### 核心逻辑

1. **参数解析**: 优先使用 task_id，否则使用 shell_id
2. **任务查找**: 在 AppState 中查找任务
3. **状态检查**: 验证任务状态为 'running'
4. **执行停止**: 调用 `stopTask()` 终止任务

## 设计要点

### 向后兼容
- 保留 `shell_id` 参数以兼容旧版 KillShell 工具
- `aliases: ['KillShell']` — 工具别名

### 并发安全
- `isConcurrencySafe(): true` — 支持并发执行

### 用户界面
- 通过 `UI.tsx` 提供自定义渲染函数

## 与其他文件的关系

### 依赖
- `../../tasks/stopTask.js` — 任务停止功能
- `../../utils/slowOperations.js` — JSON 序列化工具
- `./prompt.ts` — 提示信息
- `./UI.tsx` — UI 渲染组件

### 使用场景
- 终止长时间运行的后台任务
- 停止 shell 命令执行
- 停止智能体任务
