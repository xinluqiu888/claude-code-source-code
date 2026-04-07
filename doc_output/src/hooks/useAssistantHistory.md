# useAssistantHistory.ts — 对话历史懒加载 Hook

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/useAssistantHistory.ts`
- **类型**: React Hook
- **导出函数**: `useAssistantHistory`
- **依赖**: React useEffect/useLayoutEffect/useRef, sessionHistory

## 功能概述

本 Hook 实现 `claude assistant` 历史消息的懒加载功能：
1. 挂载时获取最新一页历史
2. 滚动到顶部附近时自动加载更旧的消息
3. 支持滚动锚定（加载后保持视口位置）
4. 自动填充视口直到内容溢出

## 核心内容详解

### 常量定义

```typescript
const PREFETCH_THRESHOLD_ROWS = 40        // 距离顶部多少行触发预加载
const MAX_FILL_PAGES = 10                 // 初始填充的最大页数
const SENTINEL_LOADING = 'loading older messages…'
const SENTINEL_LOADING_FAILED = 'failed to load older messages — scroll up to retry'
const SENTINEL_START = 'start of session'
```

### pageToMessages 转换

将历史页转换为 REPL 消息格式：
```typescript
function pageToMessages(page: HistoryPage): Message[] {
  for (const ev of page.events) {
    const c = convertSDKMessage(ev, {
      convertUserTextMessages: true,
      convertToolResults: true,
    })
    if (c.type === 'message') out.push(c.message)
  }
}
```

### 状态管理

**cursorRef**: 分页游标
- `undefined`: 初始状态，未获取第一页
- `string`: 还有更旧的消息，值为下一页 ID
- `null`: 已到达开头，无更多消息

**inflightRef**: 防止并发请求
**fillBudgetRef**: 初始填充预算，防止无限循环

### 滚动锚定

**prepend 时记录锚点**
```typescript
anchorRef.current = s
  ? { beforeHeight: s.getFreshScrollHeight(), count: msgs.length }
  : null
```

**useLayoutEffect 补偿滚动**
```typescript
const delta = s.getFreshScrollHeight() - anchor.beforeHeight
if (delta > 0) s.scrollBy(delta)
```

### 视口填充

组件挂载后检查内容高度：
```typescript
if (contentH <= viewH) {
  fillBudgetRef.current--
  void loadOlder()
}
```

## 设计要点

### 1. 哨兵消息

- 使用稳定的 UUID 复用同一个哨兵消息
- 虚拟滚动将其视为同一项（仅文本变化）
- O(1) 移除：检查 `prev[0]?.uuid === sentinelUuidRef.current`

### 2. 错误恢复

加载失败时：
1. 保留游标（不清空）
2. 显示失败提示
3. 用户下次滚动时重试

### 3. 取消处理

- 组件卸载时设置 `cancelled = true`
- 所有异步操作后检查取消状态

## 与其他文件的关系

- **sessionHistory.ts**: 提供 `fetchLatestEvents`, `fetchOlderEvents`
- **sdkMessageAdapter.ts**: 提供 `convertSDKMessage`
- **ScrollBox**: 提供滚动操作 API

## 注意事项

1. **viewerOnly 模式**: 仅在 `config.viewerOnly === true` 时启用
2. **消息过滤**: SDK 事件可能转换为非消息类型（如 tool use）
3. **高度检测**: `contentH <= viewH` 用于检测是否需要继续加载
4. **并发控制**: 使用 inflightRef 防止快速滚动触发多次请求
