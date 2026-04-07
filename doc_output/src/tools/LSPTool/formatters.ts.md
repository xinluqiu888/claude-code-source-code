# formatters.ts — LSP结果格式化器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/LSPTool/formatters.ts`
- **作用**: 格式化LSP操作结果为可读文本

## 核心内容详解

### URI格式化

```typescript
function formatUri(uri: string | undefined, cwd?: string): string
```

- 转换为相对路径（如果更短且不以`../../`开头）
- 处理URI解码（优雅降级）
- 统一使用正斜杠分隔符

### 操作格式化器

1. **formatGoToDefinitionResult**: 格式化跳转到定义结果
2. **formatFindReferencesResult**: 格式化查找引用结果，按文件分组
3. **formatHoverResult**: 格式化悬停信息
4. **formatDocumentSymbolResult**: 格式化文档符号（层级结构）
5. **formatWorkspaceSymbolResult**: 格式化工作区符号
6. **formatPrepareCallHierarchyResult**: 格式化调用层次准备结果
7. **formatIncomingCallsResult**: 格式化入站调用
8. **formatOutgoingCallsResult**: 格式化出站调用

### SymbolKind映射

```typescript
function symbolKindToString(kind: SymbolKind): string
```

将LSP的SymbolKind枚举（1-26）映射为可读字符串：
- File, Module, Namespace, Package, Class, Method, Property, Field...

## 设计要点

1. **防御性编程**: 处理undefined URI等异常情况
2. **调试日志**: 使用`logForDebugging`记录异常
3. **结果计数**: 返回结果数量和文件数量统计
4. **分组显示**: 引用结果按文件分组显示

## 与其他文件的关系

- **LSPTool.ts**: 调用格式化器处理结果
- **vscode-languageserver-types**: 使用LSP类型定义
