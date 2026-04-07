# CronCreateTool.ts — 创建Cron任务工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/ScheduleCronTool/CronCreateTool.ts`
- **作用**: 创建一次性或周期性Cron调度任务

## 功能概述

该工具允许在指定时间调度提示执行。支持标准5字段Cron表达式，可选择一次性执行（触发后自动删除）或周期性执行（7天后自动过期）。

## 核心内容详解

### 输入Schema

```typescript
z.strictObject({
  cron: z.string().describe('Standard 5-field cron expression in local time'),
  prompt: z.string().describe('The prompt to enqueue at each fire time'),
  recurring: semanticBoolean(z.boolean().optional())
    .describe('true=recurring, false=one-shot'),
  durable: semanticBoolean(z.boolean().optional())
    .describe('true=persist to disk, false=session-only'),
})
```

### 输出Schema

```typescript
z.object({
  id: z.string(),           // 任务ID
  humanSchedule: z.string(), // 人类可读的时间描述
  recurring: z.boolean(),
  durable: z.boolean().optional(),
})
```

### 验证逻辑

1. **Cron格式验证**: `parseCronExpression(input.cron)`验证5字段格式
2. **未来时间检查**: `nextCronRunMs(input.cron, Date.now())`确保在未来一年内至少匹配一次
3. **任务数量限制**: 最多50个任务
4. **队友持久化限制**: 队友不支持`durable: true`（队友会话不持久）

### 调用逻辑

```typescript
async call({ cron, prompt, recurring = true, durable = false }) {
  // kill switch强制会话级
  const effectiveDurable = durable && isDurableCronEnabled()
  const id = await addCronTask(cron, prompt, recurring, effectiveDurable, agentId)
  setScheduledTasksEnabled(true) // 启用调度器
  return { data: { id, humanSchedule, recurring, durable: effectiveDurable } }
}
```

## 设计要点

1. **延迟执行**: `shouldDefer: true`
2. **输入验证**: 4层验证（格式、时间、数量、持久化）
3. **Kill Switch**: `isDurableCronEnabled()`可强制禁用持久化
4. **自动启用**: 创建任务后自动启用调度器
5. **队友隔离**: 队友只能看到自己的任务

## 与其他文件的关系

- **cron.ts**: `cronToHuman`, `parseCronExpression`
- **cronTasks.ts**: `addCronTask`, `listAllCronTasks`
- **state.ts**: `setScheduledTasksEnabled`
- **teammateContext.ts**: `getTeammateContext`
- **UI.tsx**: 渲染函数

## 注意事项

- 任务ID由系统生成，可用于后续删除
- 周期性任务7天后自动过期
- 调度器只在REPL空闲时触发任务
- 整点任务（:00/:30）可能触发提前（最多90秒）
