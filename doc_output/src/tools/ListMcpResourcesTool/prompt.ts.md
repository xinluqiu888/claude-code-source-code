# prompt.ts — MCP资源列表提示

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/ListMcpResourcesTool/prompt.ts`
- **作用**: 定义工具名称和描述

## 核心内容详解

### 常量定义

```typescript
export const LIST_MCP_RESOURCES_TOOL_NAME = 'ListMcpResourcesTool'
```

### 工具描述

列出配置的MCP服务器中可用的资源，包含使用示例：

```typescript
export const DESCRIPTION = `
Lists available resources from configured MCP servers.
Each resource object includes a 'server' field indicating which server it's from.

Usage examples:
- List all resources from all servers: \`listMcpResources\`
- List resources from a specific server: \`listMcpResources({ server: "myserver" })\`
`
```

### 提示文本

```typescript
export const PROMPT = `
List available resources from configured MCP servers.
Each returned resource will include all standard MCP resource fields plus a 'server' field 
indicating which server the resource belongs to.

Parameters:
- server (optional): The name of a specific MCP server to get resources from...
`
```

## 与其他文件的关系

- **ListMcpResourcesTool.ts**: 导入NAME、DESCRIPTION和PROMPT
