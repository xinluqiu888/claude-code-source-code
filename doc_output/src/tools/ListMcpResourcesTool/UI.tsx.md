# UI.tsx — MCP资源列表UI

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/ListMcpResourcesTool/UI.tsx`
- **作用**: 渲染资源列表工具的消息

## 核心内容详解

### 渲染函数

1. **renderToolUseMessage**: 渲染使用消息

```typescript
export function renderToolUseMessage(
  input: Partial<{ server?: string }>
): React.ReactNode {
  return input.server 
    ? `List MCP resources from server "${input.server}"` 
    : `List all MCP resources`
}
```

2. **renderToolResultMessage**: 渲染结果消息

```typescript
export function renderToolResultMessage(
  output: Output,
  _progressMessagesForMessage: ProgressMessage<ToolProgressData>[],
  { verbose }: { verbose: boolean }
): React.ReactNode
```

### 显示逻辑

- 无结果时: 显示`(No resources found)`（灰色）
- 有结果时: JSON格式输出（美化缩进2空格）

## 与其他文件的关系

- **ListMcpResourcesTool.ts**: 导入渲染函数
- **components/MessageResponse.tsx**: 消息响应组件
- **components/shell/OutputLine.tsx**: 输出行组件
