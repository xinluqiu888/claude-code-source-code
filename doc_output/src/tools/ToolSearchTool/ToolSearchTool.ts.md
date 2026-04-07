# ToolSearchTool.ts — 工具搜索工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/ToolSearchTool/ToolSearchTool.ts`
- **类型**: TypeScript 工具模块
- **功能**: 搜索延迟加载的 Deferred Tools

## 功能概述

ToolSearchTool 允许模型通过查询获取延迟加载工具的完整模式定义。支持精确选择（select:）和关键词搜索两种模式。

## 核心内容详解

### 输入 Schema

```typescript
{
  query: string      // 搜索查询，支持 "select:Tool1,Tool2" 或关键词
  max_results?: number  // 最大结果数（默认 5）
}
```

### 输出 Schema

```typescript
{
  matches: string[]           // 匹配的工具名称列表
  query: string               // 原始查询
  total_deferred_tools: number // 延迟工具总数
  pending_mcp_servers?: string[] // 正在连接的 MCP 服务器
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `searchToolsWithKeywords()` | 关键词搜索工具 |
| `parseToolName()` | 解析工具名称为可搜索部分 |
| `compileTermPatterns()` | 预编译搜索词的正则表达式 |
| `buildSearchResult()` | 构建搜索结果结构 |
| `call()` | 执行搜索操作 |

### 搜索模式

#### 精确选择
- 格式: `select:Tool1,Tool2,Tool3`
- 支持逗号分隔的多选
- 即使工具已在已加载集合中也会返回（无害的 no-op）

#### 关键词搜索
- 支持关键词匹配
- 支持 `+required` 语法标记必需词
- 按相关性评分排序

### 评分算法

| 匹配类型 | 权重 |
|----------|------|
| MCP 工具部分精确匹配 | 12 |
| 普通工具部分精确匹配 | 10 |
| MCP 工具部分包含匹配 | 6 |
| 普通工具部分包含匹配 | 5 |
| searchHint 匹配 | 4 |
| 完整名称匹配 | 3 |
| 描述匹配 | 2 |

## 设计要点

### 缓存机制
- 使用 `memoize` 缓存工具描述
- 自动检测延迟工具变化并清除缓存

### 功能开关
- `isToolSearchEnabledOptimistic()` 控制是否启用

### 并发安全
- `isConcurrencySafe(): true` — 支持并发读取
- `isReadOnly(): true` — 不会修改状态

### 结果格式
- 返回 `tool_reference` 块用于 1P/Foundry
- 包含待连接 MCP 服务器信息

## 与其他文件的关系

### 依赖
- `../../Tool.js` — 工具基类和查找函数
- `../../utils/toolSearch.js` — 工具搜索启用检测
- `../../services/analytics/index.js` — 分析事件
- `./prompt.ts` — 工具提示信息（包含 `isDeferredTool` 等）

### 使用场景
- 模型需要调用延迟加载的工具
- 搜索特定功能的工具
- 发现 MCP 工具
