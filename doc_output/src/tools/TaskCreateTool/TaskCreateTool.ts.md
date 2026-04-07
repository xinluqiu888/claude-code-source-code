# TaskCreateTool.ts — 任务创建工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskCreateTool/TaskCreateTool.ts`
- **类型**: TypeScript 工具模块
- **功能**: 实现任务创建功能，允许用户创建结构化的任务列表来跟踪当前编码会话的进度

## 功能概述

TaskCreateTool 是一个任务管理工具，用于在任务列表中创建新任务。它支持：
- 创建带有主题、描述和进行时的任务
- 支持多智能体协作场景（队友分配）
- 执行创建任务钩子进行验证
- 自动展开任务列表视图

## 核心内容详解

### 输入 Schema

```typescript
{
  subject: string      // 任务标题（祈使形式）
  description: string  // 任务描述（需要完成的内容）
  activeForm?: string  // 进行时形式，用于显示在加载器中
  metadata?: Record    // 可选的元数据
}
```

### 输出 Schema

```typescript
{
  task: {
    id: string         // 创建的任务 ID
    subject: string    // 任务标题
  }
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `buildTool()` | 构建工具定义，包含完整的工具配置 |
| `call()` | 执行创建任务操作，包含钩子执行和验证 |
| `mapToolResultToToolResultBlockParam()` | 将结果映射为工具结果块 |

### 核心逻辑

1. **任务创建**: 调用 `createTask()` 创建新任务，初始状态为 `pending`
2. **钩子执行**: 执行 `executeTaskCreatedHooks` 进行验证
3. **错误处理**: 如果钩子返回阻塞错误，删除已创建的任务
4. **状态更新**: 自动展开任务列表视图

## 设计要点

### 并发安全
- `isConcurrencySafe: true` — 支持并发执行

### 功能开关
- 通过 `isTodoV2Enabled()` 控制是否启用

### 钩子系统
- 集成任务创建钩子，用于验证和扩展功能
- 支持阻塞错误，防止无效任务被创建

### 多智能体支持
- 检测 `isAgentSwarmsEnabled()` 以启用队友协作功能
- 任务可以分配给其他智能体

## 与其他文件的关系

### 依赖
- `../../Tool.js` — 工具基类和构建函数
- `../../utils/tasks.js` — 任务管理工具函数
- `../../utils/hooks.js` — 任务钩子执行
- `../../utils/teammate.js` — 队友信息获取
- `./constants.ts` — 工具名称常量
- `./prompt.ts` — 工具提示信息

### 被依赖
- 可能被任务管理相关的工具或 UI 组件使用

## 注意事项

1. **任务状态**: 新创建的任务状态为 `pending`，需要使用 `TaskUpdateTool` 更新状态
2. **钩子阻塞**: 如果钩子返回阻塞错误，任务会被删除，不会保留
3. **元数据**: 可以通过 `metadata` 字段附加任意元数据到任务
4. **权限**: 工具执行需要适当的权限上下文
