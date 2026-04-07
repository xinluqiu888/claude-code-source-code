# UI.tsx — ScheduleCron工具UI

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/ScheduleCronTool/UI.tsx`
- **作用**: 渲染ScheduleCron三个工具的消息（Create/Delete/List）

## 核心内容详解

### CronCreate渲染函数

1. **renderCreateToolUseMessage**: 渲染创建消息

```typescript
export function renderCreateToolUseMessage(input: Partial<{ cron: string; prompt: string }>): React.ReactNode
```

返回格式：`{cron}: {prompt}`（提示截断至60字符）

2. **renderCreateResultMessage**: 渲染创建结果

```typescript
export function renderCreateResultMessage(output: CreateOutput): React.ReactNode
```

显示：Scheduled **{id}** ({humanSchedule})

### CronDelete渲染函数

1. **renderDeleteToolUseMessage**: 渲染删除消息

```typescript
export function renderDeleteToolUseMessage(input: Partial<{ id: string }>): React.ReactNode
```

返回：`{id}`或空字符串

2. **renderDeleteResultMessage**: 渲染删除结果

```typescript
export function renderDeleteResultMessage(output: DeleteOutput): React.ReactNode
```

显示：Cancelled **{id}**

### CronList渲染函数

1. **renderListToolUseMessage**: 渲染列表消息

```typescript
export function renderListToolUseMessage(): React.ReactNode
```

返回：空字符串（列表工具无需显示输入）

2. **renderListResultMessage**: 渲染列表结果

```typescript
export function renderListResultMessage(output: ListOutput): React.ReactNode
```

显示：
- 无任务：`(No scheduled jobs)`
- 有任务：列出所有任务ID和时间表

格式：**{id}** {humanSchedule}

## 设计要点

1. **统一文件**: 三个Cron工具共享一个UI文件
2. **简洁显示**: 工具使用消息简洁，结果消息突出关键信息
3. **bold强调**: 任务ID使用bold显示
4. **dimColor**: 时间表使用dimColor弱化显示
5. **截断处理**: 长提示自动截断

## 与其他文件的关系

- **CronCreateTool.ts**: 导入CreateOutput类型
- **CronDeleteTool.ts**: 导入DeleteOutput类型
- **CronListTool.ts**: 导入ListOutput类型
- **MessageResponse.tsx**: 消息响应组件
- **format.ts**: `truncate`
