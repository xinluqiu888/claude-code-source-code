# UI.tsx — LSP工具UI渲染

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/LSPTool/UI.tsx`
- **作用**: 渲染LSP工具的使用和结果消息

## 核心内容详解

### 渲染函数

1. **userFacingName**: 返回显示名称"LSP"
2. **renderToolUseMessage**: 渲染工具使用消息
3. **renderToolUseErrorMessage**: 渲染错误消息
4. **renderToolResultMessage**: 渲染结果消息

### LSPResultSummary组件

可复用的结果摘要组件，支持折叠/展开视图：

```typescript
function LSPResultSummary({
  operation,
  resultCount,
  fileCount,
  content,
  verbose,
})
```

### 操作标签映射

```typescript
const OPERATION_LABELS: Record<Input['operation'], {
  singular: string
  plural: string
  special?: string
}>
```

为每种操作提供单复数标签：
- goToDefinition: definition/definitions
- findReferences: reference/references
- hover: hover info/hover info
- ...

### 智能上下文显示

对于位置相关操作（goToDefinition, findReferences, hover等）：
- 尝试提取位置上的符号名称
- 成功则显示: `operation: "xxx", symbol: "yyy", in: "file"`
- 失败则显示: `operation: "xxx", file: "yyy", position: line:char`

## 设计要点

1. **React.memo缓存**: 使用编译器优化渲染性能
2. **条件渲染**: verbose模式显示详细信息
3. **错误处理**: 区分verbose和非verbose错误显示
4. **CtrlOToExpand**: 可展开查看完整结果

## 与其他文件的关系

- **symbolContext.ts**: 获取符号上下文
- **components/MessageResponse.tsx**: 消息响应组件
- **components/CtrlOToExpand.tsx**: 展开组件
