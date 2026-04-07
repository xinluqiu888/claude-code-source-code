# prompt.ts — Sleep工具提示

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/SleepTool/prompt.ts`
- **作用**: 定义Sleep工具的提示文本和常量

## 核心内容详解

### 工具名称常量

```typescript
export const SLEEP_TOOL_NAME = 'Sleep'
```

### 描述常量

```typescript
export const DESCRIPTION = 'Wait for a specified duration'
```

### 提示文本常量

```typescript
export const SLEEP_TOOL_PROMPT = `Wait for a specified duration. The user can interrupt the sleep at any time.

Use this when the user tells you to sleep or rest, when you have nothing to do, or when you're waiting for something.

You may receive <${TICK_TAG}> prompts — these are periodic check-ins. Look for useful work to do before sleeping.

You can call this concurrently with other tools — it won't interfere with them.

Prefer this over \`Bash(sleep ...)\` — it doesn't hold a shell process.

Each wake-up costs an API call, but the prompt cache expires after 5 minutes of inactivity — balance accordingly.`
```

提示内容要点：
1. **用户可中断**: 用户可以随时中断sleep
2. **使用场景**:
   - 用户要求sleep或rest
   - 无事可做时
   - 等待某事时
3. **TICK提示**: 可能收到`<tick>`定期签到提示，sleep前检查是否有用工作
4. **并发执行**: 可与其他工具并发调用，不会干扰
5. **优先选择**: 优于`Bash(sleep ...)`，因为不占用shell进程
6. **成本考量**: 每次唤醒消耗API调用，但5分钟不活动后提示缓存过期，需要权衡

## 设计要点

1. **清晰说明**: 明确说明用户可中断
2. **使用指导**: 列举常见使用场景
3. **最佳实践**: 推荐该工具优于Bash sleep
4. **成本提醒**: 提醒API调用成本和缓存过期

## 与其他文件的关系

- **SleepTool.ts**: 导入使用这些常量
- **constants/xml.ts**: 使用`TICK_TAG`常量
