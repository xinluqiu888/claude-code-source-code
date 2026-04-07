# prompt.ts — Skill工具提示

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/SkillTool/prompt.ts`
- **作用**: 构建Skill工具的动态提示文本，管理技能列表预算

## 功能概述

该文件提供Skill工具的提示构建和技能列表预算管理功能。支持根据上下文窗口动态调整技能描述长度，确保技能列表在预算范围内。

## 核心内容详解

### 预算常量

```typescript
export const SKILL_BUDGET_CONTEXT_PERCENT = 0.01  // 占上下文窗口的1%
export const CHARS_PER_TOKEN = 4
export const DEFAULT_CHAR_BUDGET = 8_000  // 回退：200k tokens × 4 × 1%
export const MAX_LISTING_DESC_CHARS = 250  // 单个条目描述字符上限
```

### 预算计算

```typescript
export function getCharBudget(contextWindowTokens?: number): number
```

计算逻辑：
1. 优先使用`SLASH_COMMAND_TOOL_CHAR_BUDGET`环境变量
2. 否则根据上下文窗口计算：`tokens × 4 × 1%`
3. 回退到默认值8,000

### 描述格式化

```typescript
function getCommandDescription(cmd: Command): string
```

- 优先使用`whenToUse`，否则使用`description`
- 超过`MAX_LISTING_DESC_CHARS`则截断

### 预算内格式化

```typescript
export function formatCommandsWithinBudget(
  commands: Command[],
  contextWindowTokens?: number
): string
```

策略：
1. 首先尝试完整描述
2. 如果超预算，分区处理：
   - bundled技能（从不截断）
   - 其他技能（计算可用预算并截断）
3. 极端情况：只显示技能名称（无描述）

### 提示文本

```typescript
export const getPrompt = memoize(async (_cwd: string): Promise<string>)
```

提示内容要点：
1. 技能执行在主线对话中
2. 检查可用技能匹配用户任务
3. 用户引用"slash command"或"/<something>"时指的是技能
4. 调用示例：`skill: "pdf"`、`skill: "commit", args: "-m 'Fix bug'"`
5. **阻塞要求**: 匹配时必须先调用Skill工具再生成其他响应
6. 不提及未调用的技能
7. 不重复调用已运行的技能
8. 不用于内置CLI命令（如/help、/clear）
9. 如果已看到`<command-name>`标签，则技能已加载，直接执行

## 设计要点

1. **动态预算**: 根据上下文窗口动态计算字符预算
2. **优先级处理**: bundled技能优先，描述不截断
3. **缓存**: 使用`memoize`缓存提示文本
4. **遥测**: 超预算时记录截断事件
5. **渐进截断**: 先尝试完整描述，再截断非bundled，最后只显示名称

## 与其他文件的关系

- **SkillTool.ts**: 导入getPrompt
- **commands.ts**: `getSkillToolCommands`, `getSlashCommandToolSkills`
- **analytics/index.ts**: `logEvent`用于遥测

## 注意事项

- 技能列表用于发现，详细内容在调用时加载
- 截断发生在字符级别，不是token级别
- 截断事件用于分析技能发现效果
