# primitiveTools.ts — REPL原始工具集合

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/REPLTool/primitiveTools.ts`
- **作用**: 提供REPL模式下可用的原始工具集合

## 功能概述

该文件导出一个延迟初始化的函数，返回REPL模式下隐藏但可在REPL VM上下文中访问的基础工具列表。这些工具在REPL模式启用时从模型的直接使用中隐藏，强制模型通过REPL进行批量操作。

## 核心内容详解

### 延迟初始化函数

```typescript
export function getReplPrimitiveTools(): readonly Tool[]
```

使用延迟初始化避免"Cannot access before initialization"错误。导入链 `collapseReadSearch.ts → primitiveTools.ts → FileReadTool.tsx → ...` 会循环回到工具注册表，因此需要在调用时延迟初始化。

### 原始工具列表

包含8个基础工具：
- `FileReadTool` - 文件读取
- `FileWriteTool` - 文件写入
- `FileEditTool` - 文件编辑
- `GlobTool` - 文件搜索
- `GrepTool` - 内容搜索
- `BashTool` - Bash命令执行
- `NotebookEditTool` - Jupyter笔记本编辑
- `AgentTool` - 子代理执行

## 设计要点

1. **延迟初始化**: 避免TDZ（Temporal Dead Zone）错误
2. **循环依赖处理**: 通过函数调用而非顶层const避免初始化顺序问题
3. **工具隐藏**: REPL_ONLY_TOOLS在REPL模式下对模型隐藏
4. **直接引用**: 不通过getAllBaseTools()获取，以包含Glob/Grep（即使hasEmbeddedSearchTools()为true时）

## 与其他文件的关系

- **collapseReadSearch.ts**: 导入使用原始工具列表
- **FileReadTool.tsx等**: 被导入的基础工具
- **constants.ts**: 定义REPL_ONLY_TOOLS集合

## 注意事项

- 该文件被display-side代码使用，即使工具不在filtered execution tools列表中，也能渲染virtual messages
- 工具列表是readonly的，防止外部修改
