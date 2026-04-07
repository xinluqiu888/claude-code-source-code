# fileSuggestions.ts — 文件建议生成与索引管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/fileSuggestions.ts`
- **类型**: TypeScript 模块
- **导出函数**: `generateFileSuggestions`, `applyFileSuggestion`, `findLongestCommonPrefix`, `clearFileSuggestionCaches`, `startBackgroundCacheRefresh`, `onIndexBuildComplete`, `getDirectoryNames`, `getDirectoryNamesAsync`, `pathListSignature`

## 功能概述

本文件是 Claude Code 文件建议系统的核心模块，负责：
1. 构建和维护文件索引（使用 Rust 实现的 FileIndex）
2. 提供基于模糊匹配的文件路径建议
3. 支持 Git 仓库和非 Git 目录的文件发现
4. 实现缓存和增量更新机制
5. 处理自定义文件建议命令

## 核心内容详解

### 1. 文件索引管理

**FileIndex 单例模式**
```typescript
let fileIndex: FileIndex | null = null
function getFileIndex(): FileIndex {
  if (!fileIndex) {
    fileIndex = new FileIndex()
  }
  return fileIndex
}
```

**缓存清除函数** `clearFileSuggestionCaches()`
- 清除所有文件建议相关的缓存
- 在恢复会话时调用以确保文件发现的准确性

### 2. 路径签名机制

**pathListSignature(paths: string[]): string**
- 为路径列表生成内容哈希签名
- 采用采样策略：每 N 个路径采样一个（约 500 个样本）
- 用于检测文件列表变化，避免不必要的索引重建
- 时间复杂度：O(n/stride)，约 700 个路径只需 <1ms

### 3. 文件发现策略

**getFilesUsingGit()**
- 优先使用 `git ls-files` 获取已跟踪文件（速度快，直接读取 Git 索引）
- 后台异步获取未跟踪文件（`--others` 参数）
- 支持 `.ignore` 和 `.rgignore` 模式过滤
- 自动归一化路径到当前工作目录

**getProjectFiles()**
- Git 失败时回退到 `ripgrep --files`
- 支持 `--follow` 选项跟随符号链接
- 排除版本控制目录（.git, .svn, .hg 等）

### 4. 建议生成

**generateFileSuggestions(partialPath, showOnEmpty)**
- 支持自定义命令模式（通过 settings.json 配置）
- 空输入时返回当前目录的顶级文件/文件夹
- 处理 `./` 和 `~` 路径前缀
- 限制返回结果数量为 MAX_SUGGESTIONS (15)

**findMatchingFiles()**
- 使用 FileIndex.search() 进行模糊匹配
- 返回按相关性排序的建议列表

### 5. 建议应用

**applyFileSuggestion()**
- 将选中的建议替换到输入文本中
- 自动调整光标位置到文件路径末尾
- 支持字符串和 SuggestionItem 两种输入格式

## 设计要点

### 1. 性能优化策略

- **节流机制**: 5 秒刷新间隔，通过 `.git/index` mtime 检测变化
- **签名缓存**: 避免相同文件列表的重复索引构建
- **后台刷新**: untracked 文件获取不阻塞主流程
- **渐进式索引**: FileIndex 支持部分查询，构建过程中也可使用

### 2. 缓存层级

```
1. fileIndex: FileIndex 实例缓存
2. cachedTrackedFiles: 已跟踪文件列表
3. cachedConfigFiles: Claude 配置目录中的文件
4. cachedTrackedDirs: 目录路径缓存
5. ignorePatternsCache: 忽略模式缓存
6. loadedTrackedSignature/loadedMergedSignature: 签名缓存
```

### 3. 错误处理

- Git 命令失败自动回退到 ripgrep
- 所有异步操作都有超时控制
- 错误日志记录用于调试

## 与其他文件的关系

- **FileIndex**: 从 `src/native-ts/file-index/index.js` 导入，Rust 实现的模糊搜索索引
- **SuggestionItem**: 类型定义来自 `src/components/PromptInput/PromptInputFooterSuggestions.js`
- **executeFileSuggestionCommand**: 从 `src/utils/hooks.js` 导入，执行自定义建议命令
- **unifiedSuggestions.ts**: 将文件建议与其他类型建议（MCP 资源、Agent）统一整合

## 注意事项

1. **Git 工作树**: `.git` 是文件而非目录时（worktree），会回退到 ripgrep
2. **符号链接**: Git ls-files 不跟随符号链接（符合 Git 行为），ripgrep 使用 `--follow`
3. **大仓库优化**: 27 万+ 文件列表通过分片处理避免阻塞主线程
4. **安全性**: `pathListSignature` 故意跳过部分中间文件重命名检测，依赖 5 秒刷新兜底
5. **配置加载**: 从 `CLAUDE_CONFIG_DIRECTORIES` 加载额外配置文件用于建议
