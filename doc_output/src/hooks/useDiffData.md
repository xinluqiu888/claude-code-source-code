# useDiffData.ts — Git Diff 数据获取 Hook

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/useDiffData.ts`
- **类型**: React Hook
- **导出函数**: `useDiffData`
- **导出类型**: `DiffFile`, `DiffData`
- **依赖**: React useEffect/useState/useMemo, gitDiff utils

## 功能概述

本 Hook 获取当前 Git 仓库的 diff 数据，用于提交前的变更审查：
1. 获取 diff 统计信息和文件列表
2. 获取每个文件的 diff hunks
3. 检测大文件和截断情况
4. 识别二进制文件和未跟踪文件

## 核心内容详解

### 类型定义

```typescript
type DiffFile = {
  path: string
  linesAdded: number
  linesRemoved: number
  isBinary: boolean
  isLargeFile: boolean
  isTruncated: boolean
  isNewFile?: boolean
  isUntracked?: boolean
}

type DiffData = {
  stats: GitDiffStats | null
  files: DiffFile[]
  hunks: Map<string, StructuredPatchHunk[]>
  loading: boolean
}
```

### 数据获取

**useEffect 挂载时获取**
```typescript
useEffect(() => {
  const [statsResult, hunksResult] = await Promise.all([
    fetchGitDiff(),
    fetchGitDiffHunks(),
  ])
  setDiffResult(statsResult)
  setHunks(hunksResult)
  setLoading(false)
}, [])
```

### 文件处理逻辑

**大文件检测**
```typescript
const isLargeFile = !fileStats.isBinary && !isUntracked && !fileHunks
```
- 在统计中但无 hunk → 大文件

**截断检测**
```typescript
const totalLines = fileStats.added + fileStats.removed
const isTruncated = !isLargeFile && !fileStats.isBinary && 
                    totalLines > MAX_LINES_PER_FILE
```
- 行数超过 400 行认为已截断

### 结果计算

使用 `useMemo` 转换数据：
```typescript
return useMemo(() => {
  // 遍历 perFileStats 构建 DiffFile 列表
  // 检测大文件、截断、二进制、未跟踪
  // 按路径字母排序
  return { stats, files, hunks, loading: false }
}, [diffResult, hunks, loading])
```

## 设计要点

### 1. 并行获取

统计和 hunks 同时获取，减少等待时间

### 2. 取消支持

```typescript
let cancelled = false
// ...
return () => { cancelled = true }
```

避免组件卸载后设置状态

### 3. 文件排序

按路径字母顺序排序，保证稳定的展示顺序：
```typescript
files.sort((a, b) => a.path.localeCompare(b.path))
```

## 与其他文件的关系

- **gitDiff.ts**: 提供 `fetchGitDiff` 和 `fetchGitDiffHunks`
- **提交界面**: 使用此 Hook 显示变更摘要

## 注意事项

1. **仅挂载获取**: 不在依赖数组中，只在组件挂载时获取一次
2. **错误处理**: 出错时返回空数据和 loading=false
3. **MAX_LINES_PER_FILE = 400**: 限制单文件显示行数，避免内存问题
4. **Map 结构**: hunks 使用 Map 便于按路径快速查找
