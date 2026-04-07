# useAwaySummary.ts — 终端失焦摘要 Hook

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/useAwaySummary.ts`
- **类型**: React Hook
- **导出函数**: `useAwaySummary`
- **依赖**: React useEffect/useRef, bun:bundle feature, GrowthBook

## 功能概述

本 Hook 在终端失焦超过 5 分钟后自动生成对话摘要，帮助用户快速了解期间发生的对话内容。主要功能：
1. 监听终端焦点状态变化
2. 失焦 5 分钟后触发生成摘要
3. 避免重复生成（基于最后用户消息）
4. 支持中途取消和恢复

## 核心内容详解

### 触发条件

需要同时满足以下条件才会生成摘要：
1. 终端失焦超过 5 分钟 (`BLUR_DELAY_MS = 5 * 60_000`)
2. 当前没有进行中的对话 (`!isLoading`)
3. 自上次用户消息后未生成过摘要 (`!hasSummarySinceLastUserTurn`)
4. 功能标志已启用 (`feature('AWAY_SUMMARY')` 和 `tengu_sedge_lantern`)

### 状态检查函数

**hasSummarySinceLastUserTurn(messages)**
```typescript
// 从最新消息开始倒序遍历
for (let i = messages.length - 1; i >= 0; i--) {
  const m = messages[i]!
  if (m.type === 'user' && !m.isMeta && !m.isCompactSummary) return false
  if (m.type === 'system' && m.subtype === 'away_summary') return true
}
```

- 遇到非元数据用户消息时停止（返回 false）
- 遇到 away_summary 系统消息时返回 true

### 焦点变化处理

**onFocusChange()**

| 状态 | 行为 |
|------|------|
| 'blurred' | 清除旧定时器，设置 5 分钟延迟定时器 |
| 'focused' | 清除定时器，中止进行中的请求，重置 pending |
| 'unknown' | 无操作（终端不支持 DECSET 1004） |

### 进行中对话处理

当定时器触发时如果对话正在进行中：
1. 设置 `pendingRef.current = true`
2. 等待 `isLoading` 变为 false
3. 检查终端是否仍处于失焦状态
4. 执行摘要生成

### 摘要生成流程

1. 再次检查是否有重复摘要
2. 创建 AbortController 支持取消
3. 调用 `generateAwaySummary(messages, signal)`
4. 如果被取消或返回 null，直接返回
5. 添加 away_summary 消息到对话列表

## 设计要点

### 1. Ref 模式

使用多个 ref 避免闭包陷阱：
- `timerRef`: 定时器 ID
- `abortRef`: 中止控制器
- `messagesRef`/`isLoadingRef`: 最新状态
- `pendingRef`: 延迟生成标志
- `generateRef`: 生成函数引用

### 2. 取消机制

- 聚焦时立即中止进行中的请求
- 组件卸载时清理所有资源
- 使用 AbortSignal 支持请求级取消

### 3. 功能标志控制

双层控制：
- Bundle feature: `feature('AWAY_SUMMARY')` - 编译时控制
- GrowthBook: `tengu_sedge_lantern` - 运行时动态控制

### 4. 消息类型

生成的消息类型为：
```typescript
{
  type: 'system',
  subtype: 'away_summary',
  content: text
}
```

## 与其他文件的关系

- **terminal-focus-state.ts**: 提供 `getTerminalFocusState` 和 `subscribeTerminalFocus`
- **awaySummary.ts**: 提供 `generateAwaySummary` 服务端生成函数
- **messages.ts**: 提供 `createAwaySummaryMessage` 工厂函数
- **growthbook.ts**: 提供 GrowthBook 功能标志查询

## 注意事项

1. **DECSET 1004 支持**: 旧版终端可能不支持焦点事件报告，状态为 'unknown'
2. **5 分钟延迟**: 固定值，不可配置，平衡敏感性和实用性
3. **摘要长度**: 由 `generateAwaySummary` 控制，可能因模型而异
4. **并发安全**: 多个失焦/聚焦切换可能产生竞态条件，通过 ref 模式避免
