# QueuedMessageContext.tsx — 队列消息上下文

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/context/QueuedMessageContext.tsx`
- **类型**: React Context 组件
- **语言**: TypeScript + React

## 功能概述

为队列中的消息提供上下文，跟踪消息的队列状态、是否为第一条消息，以及容器内边距信息。用于在消息队列中正确渲染消息布局。

## 核心内容详解

### 类型定义

```typescript
type QueuedMessageContextValue = {
  isQueued: boolean    // 是否在队列中
  isFirst: boolean     // 是否是队列中的第一条
  paddingWidth: number // 容器内边距的宽度缩减值 (如 paddingX={2} 对应 4)
}
```

### 主要导出

1. **QueuedMessageProvider** — 队列消息提供者
   - 接收 `isFirst`、`useBriefLayout` 和 `children`
   - 计算并传递队列消息上下文值
   - 使用 Box 组件包裹子组件并应用内边距

2. **useQueuedMessage()** — 队列消息 hook
   - 返回 QueuedMessageContextValue 或 undefined
   - 用于访问当前消息的队列状态

### 常量

```typescript
const PADDING_X = 2  // 默认水平内边距
```

## 设计要点

1. **简写布局支持**:
   - `useBriefLayout` 标志启用简写模式
   - 简写模式下内边距为 0 (避免双重缩进)
   - HighlightedThinkingText / BriefTool UI 已通过 paddingLeft 缩进

2. **内边距计算**:
   - 标准模式: paddingX = PADDING_X (2)
   - 简写模式: paddingX = 0
   - `paddingWidth = padding * 2` 用于宽度计算

3. **上下文值记忆化**:
   - 使用 useMemo 缓存上下文值
   - 仅在 isFirst 或 padding 变化时重新计算

4. **布局包裹**:
   - 使用 Box 组件包裹子组件
   - 自动应用计算出的内边距

## 与其他文件的关系

- **../ink.js**: 提供 Box 组件
- **消息组件**: 使用 useQueuedMessage 获取队列状态
- **HighlightedThinkingText**: 使用简写布局

## 注意事项

1. 简写布局避免在 BriefTool UI 中产生双重队列缩进
2. 如果不在 Provider 内部，useQueuedMessage 返回 undefined
3. 上下文值在 isFirst 或 padding 变化时更新
4. 主要用于消息列表的视觉布局和间距控制
