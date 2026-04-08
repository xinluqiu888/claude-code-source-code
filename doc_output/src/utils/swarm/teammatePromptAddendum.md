# teammatePromptAddendum.ts — 队友系统提示追加

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/teammatePromptAddendum.ts`  
**关联模块**: 系统提示、Agent 通信  
**主要依赖**: 无

## 功能概述

本文件定义 teammate 特定的系统提示追加内容。此内容追加到完整的主 agent 系统提示后，用于 teammates。

它解释了 teammates 的可见性约束和通信要求。

## 核心内容详解

### 常量定义

```typescript
export const TEAMMATE_SYSTEM_PROMPT_ADDENDUM = `
# Agent Teammate Communication

IMPORTANT: You are running as an agent in a team. To communicate with anyone on your team:
- Use the SendMessage tool with \`to: "<name>"\` to send messages to specific teammates
- Use the SendMessage tool with \`to: "*"\` sparingly for team-wide broadcasts

Just writing a response in text is not visible to others on your team - you MUST use SendMessage tool.

The user interacts primarily with the team lead. Your work is coordinated through the task system and teammate messaging.
`
```

### 关键说明

1. **SendMessage 工具**: teammates 必须使用 `SendMessage` 工具与团队通信
2. **定向消息**: 使用 `to: "<name>"` 发送给特定 teammate
3. **广播消息**: 使用 `to: "*"` 发送团队范围广播（少用）
4. **文本不可见**: 普通文本响应对团队其他成员不可见
5. **用户交互**: 用户主要与团队领导交互

## 设计要点

1. **清晰指导**: 明确说明 teammates 必须使用工具通信
2. **团队协调**: 解释任务系统和队友消息的作用
3. **领导角色**: 强调用户主要与领导交互的架构

## 与其他文件的关系

- **inProcessRunner.ts**: 将 `TEAMMATE_SYSTEM_PROMPT_ADDENDUM` 附加到系统提示
- **system prompts**: 作为 agent 系统提示的一部分

## 注意事项

- 此追加内容仅用于 teammates，不用于主 agent
- 使用反斜杠转义模板字符串中的反引号
- 保持简洁，避免过度指导影响模型性能
