# MCPTool.ts — MCP工具执行器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/MCPTool/MCPTool.ts`
- **工具名称**: `mcp`（动态生成，实际在mcpClient.ts中覆盖）

## 功能概述

MCPTool是MCP（Model Context Protocol）工具的通用执行器。实际的工具名称、描述、提示和调用逻辑都在`mcpClient.ts`中动态覆盖。这个文件提供基础框架。

## 核心内容详解

### 基础定义

```typescript
// 允许任意输入对象，因为MCP工具定义自己的模式
export const inputSchema = lazySchema(() => z.object({}).passthrough())

// 输出为字符串
export const outputSchema = lazySchema(() =>
  z.string().describe('MCP tool execution result')
)
```

### 工具构建

```typescript
export const MCPTool = buildTool({
  isMcp: true,
  isOpenWorld() { return false },
  name: 'mcp',
  maxResultSizeChars: 100_000,
  
  // 以下在mcpClient.ts中覆盖：
  async description() { return DESCRIPTION },
  async prompt() { return PROMPT },
  async call() { return { data: '' } },
  
  // 权限检查
  async checkPermissions(): Promise<PermissionResult> {
    return {
      behavior: 'passthrough',
      message: 'MCPTool requires permission.',
    }
  },
  
  renderToolUseMessage,
  renderToolUseProgressMessage,
  renderToolResultMessage,
})
```

## 设计要点

1. **动态覆盖**: 实际行为在mcpClient.ts中定义
2. **占位符实现**: 基础实现为空，等待覆盖
3. **MCP标记**: `isMcp: true`标识这是MCP工具

## 与其他文件的关系

- **mcpClient.ts**: 覆盖工具的实际实现
- **classifyForCollapse.ts**: 分类MCP工具用于UI折叠
- **UI.tsx**: 渲染函数

## 注意事项

- 实际使用时需要通过mcpClient.ts进行配置
- 每个MCP服务器会生成多个具体工具实例
