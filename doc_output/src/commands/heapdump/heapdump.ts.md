# heapdump.ts — JavaScript堆内存导出

> **一句话总结**：导出当前Node.js进程的堆内存快照用于内存分析。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/heapdump/heapdump.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 17 行 |
| 主要职责 | 调用堆内存导出服务，将堆快照保存到桌面 |

---

## 功能概述

该文件实现了 `/heapdump` 命令，用于生成当前Node.js进程的堆内存快照（heap dump）。这个隐藏的调试命令主要用于内存泄漏分析和性能调试。

---

## 核心内容详解

### 导入与依赖

```typescript
import { performHeapDump } from '../../utils/heapDumpService.js'
```

### 主要函数

#### call(): Promise<{ type: 'text'; value: string }>

- **类型**: `async function`
- **返回值**: `Promise<{ type: 'text'; value: string }>`
- **用途**: 主入口函数

**执行流程**:

1. 调用 `performHeapDump()` 执行堆内存导出
2. 如果失败，返回包含错误信息的文本结果
3. 如果成功，返回堆文件路径和诊断文件路径

---

## 设计要点

1. **服务层抽象**: 具体的堆导出逻辑委托给 `heapDumpService`，保持命令层简洁。

2. **调试专用**: 这是一个隐藏命令，仅用于开发和调试场景。

3. **结果反馈**: 成功时返回文件路径，失败时返回错误信息。

---

## 与其他文件的关系

**依赖**:
- `performHeapDump` (`../../utils/heapDumpService.js`) - 堆导出服务

**被依赖**:
- `heapdump/index.ts` - 导出为命令配置

---

## 注意事项

1. **隐藏命令**: 配置中 `isHidden: true` 表示此命令不在帮助列表中显示。

2. **桌面保存**: 堆文件默认保存到用户桌面，便于查找。

3. **内存分析**: 生成的堆快照可以用Chrome DevTools等工具分析内存使用情况。
