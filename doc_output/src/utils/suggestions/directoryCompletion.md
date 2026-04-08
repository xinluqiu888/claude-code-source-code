# directoryCompletion.ts — 路径自动补全功能

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/suggestions/directoryCompletion.ts`  
**关联模块**: 提示输入组件、文件系统操作  
**主要依赖**: `lru-cache`, `src/utils/fsOperations.js`, `src/utils/path.js`

## 功能概述

本文件提供智能的路径自动补全功能，支持：
- 目录路径补全（仅目录）
- 文件和目录混合补全
- 基于 LRU 缓存的文件系统扫描结果缓存
- 支持相对路径（`./`, `../`）、绝对路径（`/`）和家目录路径（`~`）

## 核心内容详解

### 类型定义

```typescript
type DirectoryEntry = {
  name: string
  path: string
  type: 'directory'
}

type PathEntry = {
  name: string
  path: string
  type: 'directory' | 'file'
}

type CompletionOptions = {
  basePath?: string   // 基础路径
  maxResults?: number // 最大返回结果数，默认10
}

type PathCompletionOptions = CompletionOptions & {
  includeFiles?: boolean  // 是否包含文件，默认true
  includeHidden?: boolean // 是否包含隐藏文件，默认false
}
```

### 核心函数

#### `parsePartialPath(partialPath, basePath)`
解析部分输入路径为目录和前缀组件：
- 空输入返回当前工作目录
- 以分隔符结尾的路径视为完整目录
- 其他路径使用 `dirname` 和 `basename` 分割

#### `scanDirectory(dirPath)`
扫描目录获取子目录列表：
- 使用 LRU 缓存（500项，5分钟TTL）
- 过滤隐藏目录（以`.`开头）
- 限制最多返回100项

#### `scanDirectoryForPaths(dirPath, includeHidden)`
扫描目录获取文件和目录：
- 目录优先排序（目录排在文件前面）
- 支持包含隐藏文件选项
- 使用复合缓存键 `${dirPath}:${includeHidden}`

#### `getDirectoryCompletions(partialPath, options)`
获取纯目录补全建议：
- 返回 `SuggestionItem` 数组
- 自动在目录名后添加 `/`

#### `getPathCompletions(partialPath, options)`
获取文件和目录混合补全：
- 支持 `includeFiles` 和 `includeHidden` 选项
- 智能处理相对路径前缀（自动去除 `./`）
- 根据类型添加 `/` 后缀（仅目录）

#### `isPathLikeToken(token)`
判断字符串是否像路径（支持 `~/`, `/`, `./`, `../`, `~`, `.`, `..`）

#### `clearDirectoryCache()` / `clearPathCache()`
清除缓存函数，用于目录变更后刷新。

## 设计要点

1. **双层缓存机制**: 
   - `directoryCache`: 纯目录扫描结果缓存
   - `pathCache`: 文件+目录扫描结果缓存
   - 缓存大小：500项，TTL：5分钟

2. **路径解析策略**:
   - 使用 `expandPath` 统一处理相对路径和家目录扩展
   - 支持跨平台分隔符（`/` 和 `sep`）

3. **结果排序**: 目录优先于文件，然后按字母顺序

4. **性能优化**: 限制单次扫描最多100项，避免大目录性能问题

## 与其他文件的关系

- **fsOperations.ts**: 获取文件系统实现
- **path.ts**: 路径扩展和规范化
- **PromptInputFooterSuggestions.js**: 返回 `SuggestionItem` 类型

## 注意事项

- 扫描失败时静默返回空数组（不抛出错误）
- 路径补全会自动去除 `./` 前缀，使结果显示更简洁
- 隐藏文件过滤基于文件名是否以`.`开头，而非文件系统属性
- 缓存TTL为5分钟，对于频繁变更的目录可能需要手动清除
