# prompt.ts — ReadMcpResource工具提示

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/ReadMcpResourceTool/prompt.ts`
- **作用**: 定义ReadMcpResource工具的描述和提示文本

## 核心内容详解

### 描述常量

```typescript
export const DESCRIPTION = `
Reads a specific resource from an MCP server.
- server: The name of the MCP server to read from
- uri: The URI of the resource to read
...
`
```

### 提示文本常量

```typescript
export const PROMPT = `
Reads a specific resource from an MCP server, identified by server name and resource URI.

Parameters:
- server (required): The name of the MCP server from which to read the resource
- uri (required): The URI of the resource to read
`
```

## 设计要点

1. **简洁明确**: 描述和提示文本清晰说明工具用途
2. **参数说明**: 明确标注必需参数
3. **使用示例**: 提供实际调用示例

## 与其他文件的关系

- **ReadMcpResourceTool.ts**: 导入DESCRIPTION和PROMPT
