# prompt.ts — 笔记本编辑提示

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/NotebookEditTool/prompt.ts`
- **作用**: 工具描述和提示文本

## 核心内容详解

### 描述

```typescript
export const DESCRIPTION =
  'Replace the contents of a specific cell in a Jupyter notebook.'
```

### 提示文本

```typescript
export const PROMPT = `Completely replaces the contents of a specific cell in a Jupyter notebook (.ipynb file) with new source. Jupyter notebooks are interactive documents that combine code, text, and visualizations, commonly used for data analysis and scientific computing. The notebook_path parameter must be an absolute path, not a relative path. The cell_number is 0-indexed. Use edit_mode=insert to add a new cell at the index specified by cell_number. Use edit_mode=delete to delete the cell at the index specified by cell_number.`
```

### 关键说明

- 完全替换单元格内容
- notebook_path必须是**绝对路径**
- cell_number是**0索引**
- 支持insert和delete模式

## 与其他文件的关系

- **NotebookEditTool.ts**: 导入DESCRIPTION和PROMPT
