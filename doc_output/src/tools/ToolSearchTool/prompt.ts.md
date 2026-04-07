# prompt.ts — 工具搜索工具提示信息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/ToolSearchTool/prompt.ts`
- **类型**: TypeScript 提示信息模块
- **功能**: 定义 ToolSearch 工具的提示信息和辅助函数

## 核心内容详解

### 导出内容

```typescript
export { TOOL_SEARCH_TOOL_NAME } from './constants.js'

export function isDeferredTool(tool: Tool): boolean
export function formatDeferredToolLine(tool: Tool): string
export function getPrompt(): string
```

### isDeferredTool 函数

判断工具是否应该延迟加载：

1. **不延迟的情况**:
   - `alwaysLoad === true` — 显式选择不延迟
   - `isMcp === false` 且 `shouldDefer !== true`
   - ToolSearch 本身（模型需要它来加载其他工具）
   - Agent 工具（Fork-first 实验）
   - Brief 工具（KAIROS 功能）
   - SendUserFile 工具（KAIROS 功能且 REPL 桥激活）

2. **延迟的情况**:
   - MCP 工具（总是延迟）
   - `shouldDefer === true` 的工具

### formatDeferredToolLine 函数

格式化延迟工具为列表行：
```typescript
return tool.name  // 仅返回工具名称
```

### getPrompt 函数

返回完整的工具搜索提示信息：

#### 头部信息
- 解释工具获取延迟工具完整模式定义的用途
- 说明工具位置（system-reminder 或 available-deferred-tools）

#### 尾部信息
- 结果格式说明（`<functions>` 块中的 `<function>` 行）
- 查询形式说明：
  - `select:Read,Edit,Grep` — 精确选择
  - `notebook jupyter` — 关键词搜索
  - `+slack send` — 必需词语法

## 与其他文件的关系

### 依赖
- `../BriefTool/prompt.js` — Brief 工具名称（条件导入）
- `../SendUserFileTool/prompt.js` — SendUserFile 工具名称（条件导入）
- `../AgentTool/constants.js` — Agent 工具名称
- `../AgentTool/forkSubagent.js` — Fork 子智能体检测（延迟导入）

### 被依赖
- `ToolSearchTool.ts` — 工具主实现文件
