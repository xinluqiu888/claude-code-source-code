# compact.ts — 对话压缩命令实现

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/compact/compact.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 287 行 |
| 主要职责 | 实现对话压缩功能，清理历史但保留摘要上下文 |

## 功能概述

该文件实现了 `/compact` 命令，用于压缩对话历史。它会清理消息历史但保留一个摘要作为上下文。支持自定义摘要指令，并尝试使用会话内存压缩（如果可用）。

## 核心内容详解

### 主要函数

**call 函数**
```typescript
export const call: LocalCommandCall = async (args, context) => {
  // 获取消息（应用 compact boundary 过滤）
  messages = getMessagesAfterCompactBoundary(messages)
  
  // 尝试会话内存压缩（无自定义指令时）
  const sessionMemoryResult = await trySessionMemoryCompaction(...)
  
  // 或执行微压缩 + 传统压缩
  const microcompactResult = await microcompactMessages(messages, context)
  const result = await compactConversation(...)
}
```

### 压缩流程

1. **边界过滤**：使用 `getMessagesAfterCompactBoundary` 过滤已压缩的消息
2. **会话内存压缩**：如果没有自定义指令，优先尝试轻量级压缩
3. **微压缩**：`microcompactMessages` 减少 token 数量
4. **传统压缩**：`compactConversation` 生成摘要
5. **清理**：`runPostCompactCleanup` 清理相关状态

### 响应式压缩（REACTIVE_COMPACT）

如果启用了 `REACTIVE_COMPACT` 特性，会加载并使用响应式压缩模块：

```typescript
const reactiveCompact = feature('REACTIVE_COMPACT')
  ? require('../../services/compact/reactiveCompact.js')
  : null
```

**compactViaReactive 函数**
- 执行预压缩钩子
- 运行响应式压缩
- 合并钩子结果和压缩结果
- 返回压缩后的消息

## 设计要点

1. **分层压缩**：优先尝试轻量级的会话内存压缩
2. **自定义指令**：支持用户提供摘要生成指令
3. **钩子支持**：支持 PreCompact 和 PostCompact 钩子
4. **错误分类**：根据错误类型返回特定错误消息
5. **分析追踪**：记录压缩事件用于分析

## 与其他文件的关系

- **compact/**: 压缩服务相关模块
- **sessionMemoryCompact.ts**: 会话内存压缩
- **microCompact.ts**: 微压缩功能
- **reactiveCompact.ts**: 响应式压缩（特性开关）
- **hooks.ts**: 钩子执行
- **index.ts**: 命令注册
