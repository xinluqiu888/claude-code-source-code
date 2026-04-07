# CronDeleteTool.ts — 删除Cron任务工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/ScheduleCronTool/CronDeleteTool.ts`
- **作用**: 取消已调度的Cron任务

## 功能概述

该工具允许通过任务ID取消先前使用CronCreate创建的任务。支持删除持久化任务（从`.claude/scheduled_tasks.json`）和会话级任务（从内存存储）。

## 核心内容详解

### 输入Schema

```typescript
z.strictObject({
  id: z.string().describe('Job ID returned by CronCreate'),
})
```

### 输出Schema

```typescript
z.object({
  id: z.string(),
})
```

### 验证逻辑

1. **任务存在检查**: 在任务列表中查找指定ID
2. **所有权检查**: 队友只能删除自己的任务（非队长只能删除自己的）

```typescript
async validateInput(input): Promise<ValidationResult> {
  const task = tasks.find(t => t.id === input.id)
  if (!task) {
    return { result: false, message: `No scheduled job with id '${input.id}'`, errorCode: 1 }
  }
  const ctx = getTeammateContext()
  if (ctx && task.agentId !== ctx.agentId) {
    return { result: false, message: `Cannot delete cron job '${input.id}': owned by another agent`, errorCode: 2 }
  }
  return { result: true }
}
```

### 调用逻辑

```typescript
async call({ id }) {
  await removeCronTasks([id])
  return { data: { id } }
}
```

## 设计要点

1. **延迟执行**: `shouldDefer: true`
2. **所有权验证**: 队友只能删除自己的任务
3. **错误码**: 
   - errorCode 1: 任务不存在
   - errorCode 2: 任务属于其他代理
4. **持久化感知**: `getPath()`返回cron文件路径

## 与其他文件的关系

- **cronTasks.ts**: `listAllCronTasks`, `removeCronTasks`, `getCronFilePath`
- **teammateContext.ts**: `getTeammateContext`
- **UI.tsx**: 渲染函数
- **prompt.ts**: 工具名称和提示文本

## 注意事项

- 删除后立即生效，即使任务已接近触发时间
- 持久化任务从磁盘和内存中同时删除
- 返回的ID用于确认删除的任务
