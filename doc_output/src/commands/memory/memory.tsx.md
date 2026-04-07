# memory.tsx — 记忆文件管理

> **一句话总结**：实现交互式记忆文件选择和编辑功能。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/memory/memory.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 89 行 |
| 主要职责 | 实现记忆文件的选择、创建和编辑功能 |

---

## 功能概述

该文件实现了 `/memory` 命令的功能。提供交互式界面让用户选择记忆文件，如果文件不存在会自动创建，然后使用系统编辑器打开文件进行编辑。

---

## 核心内容详解

### 导入与依赖

```typescript
import { mkdir, writeFile } from 'fs/promises'
import * as React from 'react'
import type { CommandResultDisplay } from '../../commands.js'
import { Dialog } from '../../components/design-system/Dialog.js'
import { MemoryFileSelector } from '../../components/memory/MemoryFileSelector.js'
import { getRelativeMemoryPath } from '../../components/memory/MemoryUpdateNotification.js'
import { Box, Link, Text } from '../../ink.js'
import type { LocalJSXCommandCall } from '../../types/command.js'
import { clearMemoryFileCaches, getMemoryFiles } from '../../utils/claudemd.js'
import { getClaudeConfigHomeDir } from '../../utils/envUtils.js'
import { getErrnoCode } from '../../utils/errors.js'
import { logError } from '../../utils/log.js'
import { editFileInEditor } from '../../utils/promptEditor.js'
```

### 主要组件

#### MemoryCommand(props): React.ReactNode

- **类型**: React函数组件
- **参数**:
  - `onDone`: `(result?: string, options?: { display?: CommandResultDisplay }) => void` - 完成回调
- **用途**: 记忆文件选择和编辑的主组件

**handleSelectMemoryFile 流程**:

1. 如果路径包含Claude配置目录，递归创建目录
2. 使用 `wx` 标志创建文件（如果文件不存在），捕获 `EEXIST` 错误表示文件已存在
3. 使用 `editFileInEditor` 在系统编辑器中打开文件
4. 检测使用的编辑器环境变量（`$VISUAL` 或 `$EDITOR`）
5. 构建提示信息，告知用户当前使用的编辑器和如何更改
6. 调用 `onDone` 返回结果

#### call: LocalJSXCommandCall

- **类型**: `async function`
- **用途**: 命令入口函数

**执行流程**:

1. 清除记忆文件缓存 (`clearMemoryFileCaches`)
2. 预加载记忆文件列表 (`getMemoryFiles`) - 避免Suspense的fallback闪烁
3. 返回 `MemoryCommand` 组件

---

## 设计要点

1. **原子创建**: 使用 `wx` 标志确保文件不存在时才创建，避免覆盖现有内容。

2. **目录递归创建**: 使用 `recursive: true` 确保父目录存在。

3. **编辑器检测**: 自动检测 `$VISUAL` 和 `$EDITOR` 环境变量，告知用户当前使用的编辑器。

4. **预加载**: 在渲染前预加载记忆文件，避免Suspense的fallback闪烁。

---

## 与其他文件的关系

**依赖**:
- `Dialog` (`../../components/design-system/Dialog.js`)
- `MemoryFileSelector` (`../../components/memory/MemoryFileSelector.js`)
- `clearMemoryFileCaches`, `getMemoryFiles` (`../../utils/claudemd.js`)
- `getClaudeConfigHomeDir` (`../../utils/envUtils.js`)
- `editFileInEditor` (`../../utils/promptEditor.js`)

**被依赖**:
- `memory/index.ts` - 导出为命令配置

---

## 注意事项

1. **EEXIST处理**: 文件已存在不是错误，是正常的预期情况。

2. **编辑器优先级**: `$VISUAL` 优先于 `$EDITOR`。

3. **文档链接**: 界面提供链接到记忆功能的文档页面。
