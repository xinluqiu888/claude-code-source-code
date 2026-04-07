# UI.tsx — 网页搜索工具 UI 组件

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/WebSearchTool/UI.tsx`
- **类型**: TypeScript/React UI 模块
- **功能**: 提供 WebSearch 工具的 UI 渲染函数

## 核心内容详解

### 导出函数

```typescript
export function renderToolUseMessage(
  { query, allowed_domains, blocked_domains }: Partial<{
    query: string
    allowed_domains?: string[]
    blocked_domains?: string[]
  }>,
  { verbose }: { verbose: boolean }
): React.ReactNode

export function renderToolUseProgressMessage(
  progressMessages: ProgressMessage<WebSearchProgress>[]
): React.ReactNode

export function renderToolResultMessage(output: Output): React.ReactNode

export function getToolUseSummary(
  input: Partial<{ query: string }> | undefined
): string | null
```

### 核心逻辑

#### renderToolUseMessage
显示搜索查询：
```
"{query}"
```

Verbose 模式额外显示：
- 允许的域名列表
- 排除的域名列表

#### renderToolUseProgressMessage
支持两种进度类型：

**query_update**:
```jsx
<MessageResponse>
  <Text dimColor>Searching: {data.query}</Text>
</MessageResponse>
```

**search_results_received**:
```jsx
<MessageResponse>
  <Text dimColor>Found {data.resultCount} results for "{data.query}"</Text>
</MessageResponse>
```

#### renderToolResultMessage
显示搜索结果摘要：
```
Did {searchCount} search{es} in {timeDisplay}
```

例如：
- `Did 1 search in 1s`
- `Did 3 searches in 2.5s`

#### getSearchSummary
计算搜索统计：
- `searchCount`: 搜索结果对象数量
- `totalResultCount`: 结果总数

#### getToolUseSummary
返回截断的查询作为工具使用摘要。

## 与其他文件的关系

### 依赖
- `../../components/MessageResponse.js` — 消息响应组件
- `../../utils/format.js` — 格式化工具（截断）
- `./WebSearchTool.js` — 类型定义

### 被依赖
- `WebSearchTool.ts` — 工具主实现文件
