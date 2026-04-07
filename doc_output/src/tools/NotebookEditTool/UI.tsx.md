# UI.tsx — 笔记本编辑UI

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/NotebookEditTool/UI.tsx`
- **作用**: 渲染笔记本编辑工具的消息

## 核心内容详解

### 渲染函数

1. **getToolUseSummary**: 获取工具使用摘要

```typescript
export function getToolUseSummary(
  input: Partial<z.infer<ReturnType<typeof inputSchema>>> | undefined
): string | null {
  if (!input?.notebook_path) return null
  return getDisplayPath(input.notebook_path)
}
```

2. **renderToolUseMessage**: 渲染使用消息

```typescript
export function renderToolUseMessage(
  { notebook_path, cell_id, new_source, cell_type, edit_mode }: Partial<Input>,
  { verbose }: { verbose: boolean }
): React.ReactNode
```

- verbose模式: 显示路径@cell_id, content预览, cell_type, edit_mode
- 非verbose: 显示路径@cell_id

3. **renderToolUseRejectedMessage**: 渲染拒绝消息

使用专用组件`NotebookEditToolUseRejectedMessage`显示拒绝详情。

4. **renderToolUseErrorMessage**: 渲染错误消息

```typescript
export function renderToolUseErrorMessage(
  result: ToolResultBlockParam['content'],
  { verbose }: { verbose: boolean }
): React.ReactNode
```

5. **renderToolResultMessage**: 渲染结果消息

```typescript
export function renderToolResultMessage(
  { cell_id, new_source, error }: Output
): React.ReactNode
```

- 有错误时: 显示红色错误信息
- 成功时: 显示"Updated cell {cell_id}:" + 代码高亮

## 与其他文件的关系

- **NotebookEditTool.ts**: 导入渲染函数
- **components/FilePathLink.tsx**: 文件路径链接组件
- **components/HighlightedCode.tsx**: 代码高亮组件
- **components/NotebookEditToolUseRejectedMessage.tsx**: 拒绝消息组件
