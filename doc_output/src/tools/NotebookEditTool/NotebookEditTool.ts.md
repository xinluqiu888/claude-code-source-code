# NotebookEditTool.ts — Jupyter笔记本编辑工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/NotebookEditTool/NotebookEditTool.ts`
- **工具名称**: `NotebookEdit`

## 功能概述

编辑Jupyter笔记本（.ipynb）文件中的单元格内容。支持替换、插入和删除单元格操作。

## 核心内容详解

### 输入模式

```typescript
export const inputSchema = lazySchema(() =>
  z.strictObject({
    notebook_path: z.string(),  // 必须使用绝对路径
    cell_id: z.string().optional(),  // 单元格ID
    new_source: z.string(),     // 新内容
    cell_type: z.enum(['code', 'markdown']).optional(),
    edit_mode: z.enum(['replace', 'insert', 'delete']).optional(),
  })
)
```

### 输出模式

```typescript
const outputSchema = lazySchema(() =>
  z.object({
    new_source: z.string(),
    cell_id: z.string().optional(),
    cell_type: z.enum(['code', 'markdown']),
    language: z.string(),
    edit_mode: z.string(),
    error: z.string().optional(),
    notebook_path: z.string(),
    original_file: z.string(),  // 修改前内容（用于归因）
    updated_file: z.string(),   // 修改后内容
  })
)
```

### 验证逻辑

```typescript
async validateInput({ notebook_path, cell_type, cell_id, edit_mode = 'replace' }, toolUseContext) {
  // 1. 必须是.ipynb文件
  if (extname(fullPath) !== '.ipynb') { ... }
  
  // 2. insert模式需要cell_type
  if (edit_mode === 'insert' && !cell_type) { ... }
  
  // 3. Read-before-Edit检查
  const readTimestamp = toolUseContext.readFileState.get(fullPath)
  if (!readTimestamp) {
    return { result: false, message: 'File has not been read yet...' }
  }
  if (getFileModificationTime(fullPath) > readTimestamp.timestamp) {
    return { result: false, message: 'File has been modified since read...' }
  }
  
  // 4. 验证单元格ID存在
  // 支持实际ID或cell-N格式索引
}
```

### 执行逻辑

1. **解析单元格位置**:
   - 先尝试按实际ID查找
   - 失败则尝试解析为索引（cell-N格式）

2. **处理编辑模式**:
   - **replace**: 替换现有单元格内容
   - **insert**: 在指定单元格后插入新单元格
   - **delete**: 删除指定单元格

3. **更新单元格属性**:
   - 修改后重置代码单元格的execution_count和outputs
   - 支持更改单元格类型

4. **写入文件**:
   ```typescript
   const IPYNB_INDENT = 1
   const updatedContent = jsonStringify(notebook, null, IPYNB_INDENT)
   writeTextContent(fullPath, updatedContent, encoding, lineEndings)
   ```

## 设计要点

1. **Read-before-Edit**: 必须先读取文件才能编辑
2. **文件修改检查**: 防止在文件被外部修改后编辑
3. **归因支持**: 保存original_file和updated_file
4. **单元格ID格式**: 支持实际ID和cell-N索引
5. **文件历史**: 支持文件历史跟踪

## 与其他文件的关系

- **constants.ts**: 工具名称常量
- **prompt.ts**: 工具描述
- **UI.tsx**: 渲染函数
- **utils/notebook.ts**: 笔记本解析工具

## 注意事项

- notebook_path必须使用绝对路径
- 修改代码单元格会重置执行状态
- 支持nbformat 4.5+的单元格ID
