# memoryScan.ts — 内存扫描原语

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/memdir/memoryScan.ts`
- **类型**: TypeScript 模块
- **语言**: TypeScript

## 功能概述

内存目录扫描原语，扫描内存目录中的 .md 文件，读取其 frontmatter，并返回按时间排序的头部列表。被 `findRelevantMemories` 和 `extractMemories` 共享使用。

## 核心内容详解

### 类型定义

```typescript
export type MemoryHeader = {
  filename: string      // 相对路径文件名
  filePath: string      // 完整文件路径
  mtimeMs: number       // 修改时间 (毫秒)
  description: string | null  // frontmatter 描述
  type: MemoryType | undefined  // 内存类型
}
```

### 常量

```typescript
const MAX_MEMORY_FILES = 200       // 最大内存文件数
const FRONTMATTER_MAX_LINES = 30   // frontmatter 最大行数
```

### 主要导出

1. **scanMemoryFiles(memoryDir, signal)** — 扫描内存文件
   - 递归读取内存目录
   - 过滤 .md 文件 (排除 MEMORY.md)
   - 读取每文件的 frontmatter (前 30 行)
   - 解析 frontmatter 字段
   - 按修改时间排序 (最新的在前)
   - 限制最多 200 个文件
   - 返回 MemoryHeader 数组

2. **formatMemoryManifest(memories)** — 格式化内存清单
   - 将 MemoryHeader 数组格式化为文本清单
   - 每行格式: `- [type] filename (timestamp): description`
   - 用于选择提示词和提取工作流提示词

## 设计要点

1. **单遍扫描**:
   - readFileInRange 内部统计并返回 mtimeMs
   - 读取后排序而非先排序后读取
   - 对于 N ≤ 200 的情况，相比单独 stat 轮询减少一半系统调用

2. **错误处理**:
   - 使用 Promise.allSettled 处理单个文件读取失败
   - 失败的文件被过滤掉
   - 整个函数在异常时返回空数组

3. **AbortSignal 支持**:
   - 支持通过 signal 取消扫描
   - 传递给 readFileInRange

4. **Frontmatter 解析**:
   - 仅读取前 30 行用于 frontmatter 解析
   - 提高性能，避免读取大文件内容
   - 使用 parseFrontmatter 解析

5. **性能优化**:
   - 限制最大文件数 (200)
   - 限制 frontmatter 读取行数 (30)
   - 异步并行读取所有文件

## 与其他文件的关系

- **../utils/frontmatterParser.js**: parseFrontmatter
- **../utils/readFileInRange.js**: readFileInRange
- **./memoryTypes.js**: MemoryType, parseMemoryType
- **./findRelevantMemories.ts**: 使用 scanMemoryFiles
- **extractMemories**: 使用 scanMemoryFiles

## 注意事项

1. MEMORY.md 被排除在扫描结果外 (已在系统提示中加载)
2. 扫描是递归的，包含子目录
3. 文件读取失败不会中断整个扫描
4. 结果按修改时间排序，最新的在前
5. 最大 200 个文件限制防止大目录性能问题
