# prompt.ts — 网页搜索工具提示信息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/WebSearchTool/prompt.ts`
- **类型**: TypeScript 提示信息模块
- **功能**: 定义 WebSearch 工具的提示信息

## 核心内容详解

### 导出内容

```typescript
export const WEB_SEARCH_TOOL_NAME = 'WebSearch'

export function getWebSearchPrompt(): string
```

### getWebSearchPrompt 函数

返回完整的网页搜索提示信息，包含：

#### 功能说明
- 允许 Claude 搜索网页并使用结果响应
- 提供当前事件和最新数据
- 返回格式化为 Markdown 超链接的搜索结果
- 用于访问超出知识截止日期的信息

#### 关键要求
```
CRITICAL REQUIREMENT - You MUST follow this:
  - After answering the user's question, you MUST include a "Sources:" section
  - List all relevant URLs from the search results as markdown hyperlinks: [Title](URL)
  - This is MANDATORY - never skip including sources in your response
```

#### 使用说明
- 支持域过滤（允许/排除特定域名）
- 仅在美国可用
- 自动在单个 API 调用内执行搜索

#### 年份要求
```
IMPORTANT - Use the correct year in search queries:
  - The current month is ${currentMonthYear}.
  - You MUST use this year when searching for recent information
  - Example: If the user asks for "latest React docs", 
    search for "React documentation" with the current year, NOT last year
```

### 当前日期获取

```typescript
import { getLocalMonthYear } from 'src/constants/common.js'

const currentMonthYear = getLocalMonthYear()
// 返回格式: "April 2026"
```

## 与其他文件的关系

### 依赖
- `src/constants/common.js` — 获取当前月份年份

### 被依赖
- `WebSearchTool.ts` — 工具主实现文件
