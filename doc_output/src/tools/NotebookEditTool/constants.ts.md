# constants.ts — 笔记本编辑常量

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/NotebookEditTool/constants.ts`
- **作用**: 定义工具名称常量

## 核心内容详解

```typescript
// 单独文件以避免循环依赖
export const NOTEBOOK_EDIT_TOOL_NAME = 'NotebookEdit'
```

## 设计要点

- 单独文件避免循环依赖问题
- 用于REPLTool中的工具集合定义

## 与其他文件的关系

- **NotebookEditTool.ts**: 导入工具名称
- **REPLTool/constants.ts**: 导入到REPL_ONLY_TOOLS集合
