# prompt.ts — MCP工具提示

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/MCPTool/prompt.ts`
- **作用**: MCP工具的描述和提示（被mcpClient.ts覆盖）

## 核心内容详解

### 占位符定义

```typescript
// 实际的prompt和description在mcpClient.ts中覆盖
export const PROMPT = ''
export const DESCRIPTION = ''
```

## 设计要点

- 该文件仅提供占位符
- 实际的prompt和description由mcpClient.ts根据具体MCP服务器和工具动态生成
- 每个MCP工具实例有自己的描述和参数模式

## 与其他文件的关系

- **MCPTool.ts**: 导入这些常量（虽然为空）
- **mcpClient.ts**: 覆盖这些值
