# prompt.ts — LSP工具提示文本

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/LSPTool/prompt.ts`
- **作用**: 定义LSP工具的名称和描述文本

## 核心内容详解

### 常量定义

```typescript
export const LSP_TOOL_NAME = 'LSP' as const
```

### 工具描述

提供详细的LSP工具功能说明：

- 支持的操作列表（9种）
- 通用参数要求（filePath, line, character）
- LSP服务器配置说明

### 支持的操作

1. **goToDefinition**: 跳转到定义
2. **findReferences**: 查找引用
3. **hover**: 悬停信息
4. **documentSymbol**: 文档符号
5. **workspaceSymbol**: 工作区符号
6. **goToImplementation**: 跳转到实现
7. **prepareCallHierarchy**: 准备调用层次
8. **incomingCalls**: 入站调用
9. **outgoingCalls**: 出站调用

## 设计要点

- 清晰的1-based行列号说明
- 强调LSP服务器配置要求
- 提供使用示例

## 与其他文件的关系

- **LSPTool.ts**: 主工具实现，导入NAME和DESCRIPTION
