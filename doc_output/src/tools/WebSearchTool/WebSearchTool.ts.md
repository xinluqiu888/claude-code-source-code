# WebSearchTool.ts — 网页搜索工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/WebSearchTool/WebSearchTool.ts`
- **类型**: TypeScript 工具模块
- **功能**: 搜索网页获取最新信息

## 功能概述

WebSearchTool 允许 Claude 搜索网页并使用结果来响应用户。支持域过滤，结果格式化为 Markdown 超链接。

## 核心内容详解

### 输入 Schema

```typescript
{
  query: string                    // 搜索查询
  allowed_domains?: string[]       // 仅包含这些域名的结果
  blocked_domains?: string[]       // 排除这些域名的结果
}
```

### 输出 Schema

```typescript
{
  query: string                    // 执行的搜索查询
  results: (SearchResult | string)[] // 搜索结果和/或文本评论
  durationSeconds: number          // 搜索耗时
}

// SearchResult
{
  tool_use_id: string
  content: Array<{ title: string; url: string }>
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `makeToolSchema()` | 创建搜索工具模式 |
| `makeOutputFromSearchResponse()` | 从搜索响应构建输出 |
| `call()` | 执行搜索操作 |
| `mapToolResultToToolResultBlockParam()` | 格式化结果为工具结果块 |

### 核心逻辑

1. **创建用户消息**: 构建搜索请求消息
2. **创建工具模式**: 使用 `makeToolSchema()` 创建搜索工具
3. **模型查询**: 调用带搜索工具能力的模型
4. **处理响应**: 解析内容块序列
5. **格式化输出**: 将结果格式化为带超链接的文本

### 进度跟踪

支持两种进度类型：
- `query_update`: 搜索查询更新
- `search_results_received`: 收到搜索结果

## 设计要点

### 功能开关

```typescript
isEnabled() {
  // firstParty: 始终启用
  // Vertex AI: Claude 4.0+ 模型
  // Foundry: 始终启用
}
```

### 并发和只读

- `isConcurrencySafe(): true` — 支持并发
- `isReadOnly(): true` — 不会修改状态

### 权限检查

```typescript
checkPermissions() {
  return {
    behavior: 'passthrough',
    message: 'WebSearchTool requires permission.',
    suggestions: [...]
  }
}
```

### 输入验证

- 查询不能为空
- 不能同时指定 `allowed_domains` 和 `blocked_domains`

### 搜索限制

- 最大搜索次数: 8 次

## 与其他文件的关系

### 依赖
- `@anthropic-ai/sdk` — SDK 类型
- `../../services/api/claude.js` — 模型查询
- `../../utils/model/model.js` — 模型获取
- `../../utils/messages.js` — 消息创建
- `./prompt.ts` — 搜索提示信息
- `./UI.tsx` — UI 渲染组件

### 使用场景
- 获取当前事件和最新数据
- 访问超出 Claude 知识截止日期的信息
- 搜索文档和技术信息
