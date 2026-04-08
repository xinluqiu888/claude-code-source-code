# slackChannelSuggestions.ts — Slack 频道名称智能补全

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/suggestions/slackChannelSuggestions.ts`  
**关联模块**: MCP 服务、提示输入组件  
**主要依赖**: `zod`, `src/services/mcp/types.js`, `src/utils/signal.js`, `src/utils/slowOperations.js`

## 功能概述

本文件实现 Slack 频道名称的智能补全功能：
- 通过 MCP 服务器搜索 Slack 频道
- 本地缓存和复用搜索结果
- 频道名称高亮标记（仅确认存在的频道显示蓝色）
- 智能查询优化（处理分词和搜索词截断）

## 核心内容详解

### 常量与状态

```typescript
const SLACK_SEARCH_TOOL = 'slack_search_channels'

// 缓存状态
const cache = new Map<string, string[]>()           // 查询结果缓存
const knownChannels = new Set<string>()             // 已知频道集合
let knownChannelsVersion = 0                        // 版本号，用于变更通知
const knownChannelsChanged = createSignal()         // 变更信号

// 进行中查询状态
let inflightQuery: string | null = null
let inflightPromise: Promise<string[]> | null = null
```

### 核心函数

#### `fetchChannels(clients, query)`
从 MCP 服务器获取频道列表：
- 查找 slack 类型的 MCP 客户端
- 调用 `slack_search_channels` 工具
- 解析返回的 Markdown 格式结果
- 5秒超时保护

#### `parseChannels(text)`
解析 Slack MCP 返回的频道列表：
- 从 Markdown 中提取 `Name: #channel-name` 行
- 使用正则 `/^Name:\s*#?([a-z0-9][a-z0-9_-]{0,79})\s*$/` 匹配有效频道名
- 返回去重的频道名称数组

#### `mcpQueryFor(searchToken)`
优化 MCP 查询词：
- 截断最后一个分隔符后的部分（`-` 或 `_`）
- Slack 搜索按分隔符分词，部分词会导致0结果
- 本地过滤保留完整匹配逻辑

#### `findReusableCacheEntry(mcpQuery, searchToken)`
查找可复用的缓存条目：
- 遍历缓存查找当前查询的前缀匹配项
- 确保缓存结果包含以 `searchToken` 开头的频道
- 返回最长匹配键的缓存

#### `getSlackChannelSuggestions(clients, searchToken)`
获取频道补全建议：
- 优先从缓存获取
- 复用进行中查询避免重复请求
- 更新 `knownChannels` 集合并触发变更信号
- 限制缓存大小为50项
- 返回前10个匹配项

#### `findSlackChannelPositions(text)`
查找文本中 Slack 频道位置：
- 正则 `/(^|\s)#([a-z0-9][a-z0-9_-]{0,79})(?=\s|$)/g`
- 仅标记存在于 `knownChannels` 中的频道
- 返回 `{start, end}` 位置数组

### 辅助函数

#### `unwrapResults(text)`
解包 Slack MCP 返回的 JSON 信封：
- 有些响应包装在 `{"results": "..."}` 中
- 使用 `zod` 安全解析

#### `hasSlackMcpServer(clients)`
检查是否配置了 Slack MCP 服务器。

#### `clearSlackChannelCache()`
清除所有缓存和状态。

## 设计要点

1. **智能缓存策略**:
   - 前缀复用：`c` 的查询结果可用于 `cl` 的补全
   - 进行中查询共享：避免同一词的并发请求
   - 50项缓存限制，LRU淘汰

2. **查询优化**:
   - 截断不完整词避免0结果
   - 本地过滤确保精确匹配
   - 20结果限制，通过精确查询词规避

3. **频道验证**:
   - `knownChannels` 集合存储确认存在的频道
   - 版本号+信号机制通知 UI 更新高亮
   - 防止虚假频道名高亮

4. **频道名规范**:
   - 小写字母、数字、下划线、连字符
   - 长度1-80字符
   - 必须以字母数字开头

## 与其他文件的关系

- **mcp/types.ts**: MCP 服务器连接类型
- **signal.ts**: 信号机制用于频道集合变更通知
- **slowOperations.ts**: 提供 `jsonParse` 工具
- **PromptInputFooterSuggestions.js**: 返回 `SuggestionItem` 类型

## 注意事项

- Slack MCP 服务器必须已连接且包含 "slack" 名称
- 搜索有5秒超时，超时返回空结果
- 频道名匹配不区分大小写，但存储为小写
- 正则匹配需符合 Slack 频道命名规范
- 结果信封解析失败时回退到原始文本
