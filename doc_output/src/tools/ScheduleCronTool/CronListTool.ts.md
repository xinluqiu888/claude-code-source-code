# CronListTool.ts — 列出Cron任务工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/ScheduleCronTool/CronListTool.ts`
- **作用**: 列出所有已调度的Cron任务

## 功能概述

该工具显示当前会话中所有活动的Cron任务，包括持久化任务（从`.claude/scheduled_tasks.json`）和会话级任务（内存中）。队友只能看到自己的任务，队长可以看到所有任务。

## 核心内容详解

### 输入Schema

```typescript
z.strictObject({}) // 无输入参数
```

### 输出Schema

```typescript
z.object({
  jobs: z.array(
    z.object({
      id: z.string(),
      cron: z.string(),
      humanSchedule: z.string(),
      prompt: z.string(),
      recurring: z.boolean().optional(),
      durable: z.boolean().optional(),
    })
  ),
})
```

### 调用逻辑

```typescript
async call() {
  const allTasks = await listAllCronTasks()
  // 队友过滤：只看到自己的任务
  const ctx = getTeammateContext()
  const tasks = ctx
    ? allTasks.filter(t => t.agentId === ctx.agentId)
    : allTasks
  const jobs = tasks.map(t => ({
    id: t.id,
    cron: t.cron,
    humanSchedule: cronToHuman(t.cron),
    prompt: t.prompt,
    ...(t.recurring ? { recurring: true } : {}),
    ...(t.durable === false ? { durable: false } : {}),
  }))
  return { data: { jobs } }
}
```

### 结果格式化

```typescript
mapToolResultToToolResultBlockParam(output, toolUseID) {
  return output.jobs.length > 0
    ? output.jobs
        .map(j =>
          `${j.id} — ${j.humanSchedule}${j.recurring ? ' (recurring)' : ' (one-shot)'}${j.durable === false ? ' [session-only]' : ''}: ${truncate(j.prompt, 80, true)}`
        )
        .join('\n')
    : 'No scheduled jobs.'
}
```

## 设计要点

1. **只读操作**: `isReadOnly() => true`
2. **并发安全**: `isConcurrencySafe() => true`
3. **延迟执行**: `shouldDefer: true`
4. **权限隔离**: 队友只能看到自己的任务
5. **人类可读时间**: `cronToHuman`转换Cron为人类可读格式
6. **提示截断**: 最多显示80字符

## 与其他文件的关系

- **cron.ts**: `cronToHuman`
- **cronTasks.ts**: `listAllCronTasks`
- **teammateContext.ts**: `getTeammateContext`
- **format.ts**: `truncate`
- **UI.tsx**: 渲染函数

## 注意事项

- 队长可以看到所有任务
- 队友只能看到分配给自己的任务
- `[session-only]`标记表示非持久化任务
- `(recurring)`标记表示周期性任务，`(one-shot)`表示一次性任务
