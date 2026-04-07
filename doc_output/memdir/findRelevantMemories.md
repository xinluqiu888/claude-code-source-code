# findRelevantMemories.ts — 查找相关内存

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/memdir/findRelevantMemories.ts`
- **类型**: TypeScript 模块
- **语言**: TypeScript

## 功能概述

通过扫描内存文件头部并询问 Sonnet 选择最相关的文件来查找与查询相关的内存。支持工具使用过滤和已展示文件过滤。

## 核心内容详解

### 类型定义

```typescript
export type RelevantMemory = {
  path: string    // 绝对文件路径
  mtimeMs: number // 修改时间
}
```

### 常量

```typescript
const SELECT_MEMORIES_SYSTEM_PROMPT = `...`  // 内存选择系统提示词
```

### 主要导出

**findRelevantMemories(query, memoryDir, signal, recentTools?, alreadySurfaced?)**

参数:
- `query`: 用户查询字符串
- `memoryDir`: 内存目录路径
- `signal`: AbortSignal 用于取消
- `recentTools`: 最近使用的工具列表 (可选，默认 [])
- `alreadySurfaced`: 已展示文件路径集合 (可选，默认空 Set)

返回: RelevantMemory[] (最多 5 个)

### 处理流程

1. **扫描内存文件**:
   - 调用 scanMemoryFiles 获取头部列表
   - 过滤掉 alreadySurfaced 中的文件
   - 若无文件返回空数组

2. **选择相关内存**:
   - 使用 sideQuery 调用 Sonnet
   - 提供用户查询和可用内存清单
   - 请求 JSON 格式响应

3. **处理结果**:
   - 解析 selected_memories 数组
   - 过滤有效文件名
   - 映射回 MemoryHeader

4. **遥测记录**:
   - 如果 MEMORY_SHAPE_TELEMETRY 特性启用
   - 记录选择率和年龄分布

5. **返回结果**:
   - 返回选定文件的路径和修改时间
   - mtime 用于新鲜度提示

### 工具过滤

当提供 recentTools 时:
- 添加到提示词作为上下文
- 防止选择工具引用文档的误报
- 仍选择包含警告和已知问题的内存

## 设计要点

1. **最多 5 个结果**:
   - 限制上下文大小
   - 鼓励选择性

2. **已展示过滤**:
   - alreadySurfaced 过滤之前回合展示的文件
   - 选择器预算用于新候选

3. **JSON Schema 输出**:
   - 强制结构化响应
   - 减少解析错误

4. **错误处理**:
   - signal 取消时返回空数组
   - 解析失败时返回空数组
   - 记录调试日志

5. **遥测**:
   - 记录选择率 (需要分母)
   - 记录年龄分布 (-1 表示未运行)

6. **动态导入**:
   - MEMORY_SHAPE_TELEMETRY 特性使用 require 动态导入
   - 避免不必要地加载遥测模块

## 与其他文件的关系

- **./memoryScan.ts**: scanMemoryFiles, formatMemoryManifest
- **../utils/sideQuery.ts**: sideQuery 函数
- **../utils/model/model.ts**: getDefaultSonnetModel
- **./memoryShapeTelemetry.ts**: logMemoryRecallShape (动态导入)

## 注意事项

1. 排除 MEMORY.md (已在系统提示中)
2. 返回 mtime 以便调用者显示新鲜度
3. 空选择也记录遥测 (选择率需要分母)
4. 最近工具用于减少误报
5. 已展示文件在调用前过滤
