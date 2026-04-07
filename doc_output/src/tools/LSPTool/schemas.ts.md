# schemas.ts — LSP输入验证模式

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/LSPTool/schemas.ts`
- **作用**: 定义LSP工具的Zod验证模式

## 核心内容详解

### 模式设计

使用**可区分的联合类型**（Discriminated Union），以`operation`作为区分字段：

```typescript
export const lspToolInputSchema = lazySchema(() => {
  // 9种操作模式，每种都有相同的基字段
  const goToDefinitionSchema = z.strictObject({
    operation: z.literal('goToDefinition'),
    filePath: z.string(),
    line: z.number().int().positive(),
    character: z.number().int().positive(),
  })
  // ... 其他8种操作

  return z.discriminatedUnion('operation', [
    goToDefinitionSchema,
    findReferencesSchema,
    hoverSchema,
    documentSymbolSchema,
    workspaceSymbolSchema,
    goToImplementationSchema,
    prepareCallHierarchySchema,
    incomingCallsSchema,
    outgoingCallsSchema,
  ])
})
```

### 类型导出

```typescript
export type LSPToolInput = z.infer<ReturnType<typeof lspToolInputSchema>>

// 类型守卫
export function isValidLSPOperation(operation: string): boolean
```

## 设计要点

1. **lazySchema**: 延迟加载避免循环依赖
2. **strictObject**: 严格模式，禁止额外字段
3. **1-based索引**: 行列号使用正整数，与编辑器一致
4. **类型守卫**: 提供运行时操作名验证

## 与其他文件的关系

- **LSPTool.ts**: 使用模式进行输入验证
- **validateInput**: 使用`lspToolInputSchema`进行类型安全验证
