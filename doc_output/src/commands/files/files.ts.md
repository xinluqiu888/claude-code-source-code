# files.ts — 列出上下文中的文件

> **一句话总结**：列出当前REPL上下文中所有已加载的文件。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/files/files.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 19 行 |
| 主要职责 | 显示当前上下文中已加载的所有文件列表 |

---

## 功能概述

该文件实现了 `/files` 命令，用于显示当前REPL会话上下文中已加载的所有文件。这是内部命令（仅'ant'用户类型可用），主要用于调试和开发目的。

---

## 核心内容详解

### 导入与依赖

```typescript
import { relative } from 'path'
import type { ToolUseContext } from '../../Tool.js'
import type { LocalCommandResult } from '../../types/command.js'
import { getCwd } from '../../utils/cwd.js'
import { cacheKeys } from '../../utils/fileStateCache.js'
```

### 主要函数

#### call(_args, context): Promise<LocalCommandResult>

- **类型**: `async function`
- **参数**:
  - `_args`: `string` - 命令参数（未使用）
  - `context`: `ToolUseContext` - 工具使用上下文
- **返回值**: `Promise<LocalCommandResult>`
- **用途**: 主入口函数

**执行流程**:

1. 从上下文中获取文件列表：`context.readFileState ? cacheKeys(context.readFileState) : []`
2. 如果文件列表为空，返回 "No files in context"
3. 使用 `relative(getCwd(), file)` 将绝对路径转为相对路径
4. 将文件列表用换行符连接，返回格式化的文本结果

---

## 设计要点

1. **调试工具**: 这是一个内部调试命令，帮助开发者了解当前上下文中加载了哪些文件。

2. **路径简化**: 使用相对路径显示，避免过长的绝对路径影响可读性。

3. **状态缓存**: 使用 `cacheKeys` 从文件状态缓存中提取文件列表。

4. **简洁输出**: 纯文本列表格式，易于阅读和复制。

---

## 与其他文件的关系

**依赖**:
- `getCwd` (`../../utils/cwd.js`) - 获取当前工作目录
- `cacheKeys` (`../../utils/fileStateCache.js`) - 从缓存中提取键

**被依赖**:
- `files/index.ts` - 导出为命令配置

---

## 注意事项

1. **内部命令**: `isEnabled: () => process.env.USER_TYPE === 'ant'` 表明这是内部调试命令。

2. **非交互式支持**: `supportsNonInteractive: true` 允许在非交互式环境中使用。

3. **ToolUseContext**: 依赖ToolUseContext的readFileState属性获取文件信息。
