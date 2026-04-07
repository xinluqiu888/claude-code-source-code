# unifiedSuggestions.ts — 统一建议生成器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/unifiedSuggestions.ts`
- **类型**: TypeScript 模块
- **导出函数**: `generateUnifiedSuggestions`
- **依赖**: fuse.js, path, fileSuggestions, agentColorManager

## 功能概述

本模块将多种类型的建议源（文件、MCP 资源、Agent）整合为统一的建议列表，提供：
1. 多源建议的聚合与排序
2. 基于 Fuse.js 的模糊搜索
3. 加权评分系统（不同字段不同权重）
4. 统一的数据格式转换

## 核心内容详解

### 建议源类型定义

```typescript
type FileSuggestionSource = {
  type: 'file'
  displayText: string
  description?: string
  path: string
  filename: string
  score?: number  // nucleo 评分（0-1，越小越好）
}

type McpResourceSuggestionSource = {
  type: 'mcp_resource'
  displayText: string
  description: string
  server: string
  uri: string
  name: string
}

type AgentSuggestionSource = {
  type: 'agent'
  displayText: string
  description: string
  agentType: string
  color?: keyof Theme  // 关联颜色主题
}
```

### 核心函数: generateUnifiedSuggestions

**参数**
- `query`: 搜索查询字符串
- `mcpResources`: MCP 服务器资源记录
- `agents`: Agent 定义列表
- `showOnEmpty`: 空查询时是否显示建议

**处理流程**

1. **并行获取建议源**
   ```typescript
   const [fileSuggestions, agentSources] = await Promise.all([
     generateFileSuggestions(query, showOnEmpty),
     Promise.resolve(generateAgentSuggestions(agents, query, showOnEmpty)),
   ])
   ```

2. **文件建议转换**
   - 使用 `basename` 提取文件名
   - 复用 fileSuggestions 返回的 score（来自 nucleo 模糊匹配）

3. **MCP 资源转换**
   - 展平所有服务器的资源列表
   - 截断描述文本至 60 字符

4. **Agent 建议生成**
   - 空查询时返回所有 Agent
   - 非空查询时进行简单字符串匹配

5. **混合评分与排序**
   - 文件建议：使用 nucleo 评分（已排序）
   - MCP/Agent：使用 Fuse.js 评分
   - 合并后按 score 升序排列

### 评分权重配置

```typescript
keys: [
  { name: 'displayText', weight: 2 },
  { name: 'name', weight: 3 },
  { name: 'server', weight: 1 },
  { name: 'description', weight: 1 },
  { name: 'agentType', weight: 3 },
]
```

- `name` 和 `agentType` 权重最高（3）
- `displayText` 次之（2）
- `server` 和 `description` 权重最低（1）

## 设计要点

### 1. 双引擎策略

- **文件**: 使用 Rust nucleo 引擎（通过 FileIndex），速度快、质量高
- **其他**: 使用 Fuse.js，纯 JavaScript，灵活性高

### 2. 评分归一化

- nucleo 和 Fuse.js 都返回 0-1 范围的分数
- 两者可以混合比较排序
- 缺失 score 时默认使用 0.5

### 3. 空查询处理

- `showOnEmpty=false`: 返回空数组
- `showOnEmpty=true`: 返回所有源的前 N 个（MAX_UNIFIED_SUGGESTIONS = 15）

### 4. 描述截断

- 使用 `truncateToWidth` 处理多字节字符
- 限制为 60 字符宽度（非字节数）

## 与其他文件的关系

- **fileSuggestions.ts**: 提供文件建议生成
- **fuse.js**: 模糊搜索库，用于非文件源
- **agentColorManager.ts**: 提供 Agent 颜色映射
- **SuggestionItem**: 类型来自 PromptInputFooterSuggestions
- **format.ts**: 提供 truncateToWidth 工具函数

## 注意事项

1. **评分阈值**: Fuse.js 使用 0.6 阈值，允许更多匹配通过，依赖后续排序筛选
2. **性能**: 文件建议使用 Rust 索引，其他源使用 Fuse.js，两者都是 O(n) 或更优
3. **Agent 匹配简单**: Agent 建议只使用简单字符串包含匹配，不使用 Fuse.js
4. **MCP 资源**: 来自所有服务器的资源被展平处理，无服务器优先级概念
