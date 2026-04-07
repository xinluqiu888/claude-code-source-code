# grouping.ts — 按 API 轮次分组消息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/compact/grouping.ts`
- **作用域**: 消息分组逻辑
- **主要导出**:
  - `groupMessagesByApiRound`: 按 API 轮次分组消息

## 功能概述

将消息按 API 轮次（round-trip）分组。一个边界在助手响应开始时触发（与前一个助手不同的 `message.id`）。这允许反应式压缩在单提示代理会话（SDK/CCR/eval 调用者）上操作。

## 核心内容详解

### 分组逻辑

```typescript
export function groupMessagesByApiRound(messages: Message[]): Message[][] {
  const groups: Message[][] = []
  let current: Message[] = []
  let lastAssistantId: string | undefined

  for (const msg of messages) {
    if (
      msg.type === 'assistant' &&
      msg.message.id !== lastAssistantId &&
      current.length > 0
    ) {
      groups.push(current)
      current = [msg]
    } else {
      current.push(msg)
    }
    if (msg.type === 'assistant') {
      lastAssistantId = msg.message.id
    }
  }

  if (current.length > 0) {
    groups.push(current)
  }
  return groups
}
```

### 边界判定

- 助手消息且 `message.id` 与前一个不同
- 同一助手响应的流式块共享相同的 `message.id`
- 边界只在新的 API 轮次开始时触发

### 工具对处理

- 良好格式化的对话：API 保证每个 `tool_use` 在下一个助手轮次前被解决
- 格式错误的输入（恢复/截断后的悬空 `tool_use`）：fork 的 `ensureToolResultPairing` 在 API 时间修复

## 设计要点

1. **API 安全分割点**: 基于助手 ID 的边界是 API 安全的分割点
2. **细粒度分组**: 比人类轮次分组更细粒度
3. **单提示会话支持**: 支持代理会话的细粒度操作
4. **循环打破**: 提取到独立文件以打破 `compact.ts` 和 `compactMessages.ts` 之间的循环

## 与其他文件的关系

- **compact.ts**: 使用 `groupMessagesByApiRound` 进行 PTL 重试截断
- **reactiveCompact.ts**: 可能使用此分组

## 注意事项

1. **格式假设**: 假设对话格式良好，API 会保证工具对完整
2. **悬空工具**: 对格式不良的输入（悬空 `tool_use`）由 fork 层修复
3. **提取原因**: 该文件提取以打破模块初始化顺序导致的循环依赖
