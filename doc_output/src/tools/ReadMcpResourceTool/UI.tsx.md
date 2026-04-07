# UI.tsx — ReadMcpResource工具UI

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/ReadMcpResourceTool/UI.tsx`
- **作用**: 渲染ReadMcpResource工具的消息

## 核心内容详解

### 渲染函数

1. **renderToolUseMessage**: 渲染工具使用消息

```typescript
export function renderToolUseMessage(
  input: Partial<z.infer<ReturnType<typeof inputSchema>>>
): React.ReactNode
```

返回格式：`Read resource "{uri}" from server "{server}"`

2. **userFacingName**: 用户可见名称

```typescript
export function userFacingName(): string {
  return 'readMcpResource'
}
```

3. **renderToolResultMessage**: 渲染结果消息

```typescript
export function renderToolResultMessage(
  output: Output,
  _progressMessagesForMessage: ProgressMessage<ToolProgressData>[],
  { verbose }: { verbose: boolean }
): React.ReactNode
```

显示逻辑：
- 无内容：显示`(No content)`
- 有内容：以JSON格式美化输出

## 设计要点

1. **简洁显示**: 工具使用消息显示服务器和URI
2. **JSON格式化**: 结果以格式化的JSON显示
3. **空内容处理**: 区分无输出和正常输出

## 与其他文件的关系

- **ReadMcpResourceTool.ts**: 导入inputSchema和Output类型
- **MessageResponse.tsx**: 消息响应组件
- **OutputLine.tsx**: 输出行组件
