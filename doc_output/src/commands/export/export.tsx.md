# export.tsx — 导出对话内容到文件

> **一句话总结**：实现对话导出功能，支持直接写入文件或弹出导出对话框。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/export/export.tsx` |
| 文件类型 | TypeScript React (TSX) |
| 代码行数 | 90 行 |
| 主要职责 | 将当前对话内容导出为文本文件 |

---

## 功能概述

该文件实现了 `/export [filename]` 命令，用于导出当前REPL会话的对话内容。支持两种模式：

1. **直接导出模式**：如果用户提供了文件名参数，直接将内容写入该文件。
2. **交互式导出模式**：如果没有提供文件名，显示ExportDialog组件让用户选择导出位置和方式。

文件内容格式化为纯文本，使用React渲染器处理消息内容。文件名可以基于对话的第一条用户提示自动生成。

---

## 核心内容详解

### 导入与依赖

```typescript
import { join } from 'path'
import React from 'react'
import { ExportDialog } from '../../components/ExportDialog.js'
import type { ToolUseContext } from '../../Tool.js'
import type { LocalJSXCommandOnDone } from '../../types/command.js'
import type { Message } from '../../types/message.js'
import { getCwd } from '../../utils/cwd.js'
import { renderMessagesToPlainText } from '../../utils/exportRenderer.js'
import { writeFileSync_DEPRECATED } from '../../utils/slowOperations.js'
```

### 主要函数

#### formatTimestamp(date: Date): string

- **类型**: `function`
- **参数**: `date` - 日期对象
- **返回值**: `string` - 格式化的时间戳字符串
- **用途**: 生成 `YYYY-MM-DD-HHMMSS` 格式的时间戳

#### extractFirstPrompt(messages: Message[]): string

- **类型**: `function`
- **参数**: `messages` - 消息数组
- **返回值**: `string` - 提取的第一条用户提示
- **用途**: 从对话中提取第一条用户消息作为文件名基础
- **关键逻辑**:
  - 查找第一条 `type === 'user'` 的消息
  - 支持字符串和数组两种content格式
  - 只取第一行，限制50字符长度

#### sanitizeFilename(text: string): string

- **类型**: `function`
- **参数**: `text` - 原始文本
- **返回值**: `string` - 清理后的文件名安全字符串
- **用途**: 将文本转换为合法文件名
- **关键逻辑**:
  - 转小写
  - 移除特殊字符
  - 空格替换为连字符
  - 移除首尾连字符

#### exportWithReactRenderer(context: ToolUseContext): Promise<string>

- **类型**: `async function`
- **参数**: `context` - 工具使用上下文
- **返回值**: `Promise<string>` - 渲染后的纯文本内容
- **用途**: 使用React渲染器将消息转换为纯文本

#### call(onDone, context, args): Promise<React.ReactNode>

- **类型**: `async function`
- **参数**:
  - `onDone`: `LocalJSXCommandOnDone` - 完成回调
  - `context`: `ToolUseContext` - 工具上下文
  - `args`: `string` - 用户提供的文件名参数
- **返回值**: `Promise<React.ReactNode>`
- **用途**: 主入口函数

**执行流程**:

1. 渲染对话内容为纯文本
2. 如果提供了文件名参数：
   - 确保文件扩展名为 `.txt`
   - 写入文件到当前工作目录
   - 调用 `onDone` 返回成功/失败消息
3. 如果没有提供文件名：
   - 提取第一条提示生成默认文件名
   - 返回 `ExportDialog` 组件供用户交互

---

## 设计要点

1. **双模式支持**: 同时支持命令行直接导出和交互式对话框，满足不同用户习惯。

2. **智能文件名生成**: 基于对话内容自动生成描述性文件名，提高文件可识别性。

3. **安全文件名处理**: `sanitizeFilename` 确保生成的文件名在所有文件系统上都合法。

4. **React渲染复用**: 使用 `renderMessagesToPlainText` 复用现有的消息渲染逻辑。

---

## 与其他文件的关系

**依赖**:
- `ExportDialog` 组件 (`../../components/ExportDialog.js`) - 导出对话框UI
- `getCwd` (`../../utils/cwd.js`) - 获取当前工作目录
- `renderMessagesToPlainText` (`../../utils/exportRenderer.js`) - 消息渲染器
- `writeFileSync_DEPRECATED` (`../../utils/slowOperations.js`) - 文件写入

**被依赖**:
- `export/index.ts` - 导出为命令配置

---

## 注意事项

1. **DEPRECATED函数**: 使用 `writeFileSync_DEPRECATED` 标记该函数将在未来版本中替换。

2. **文件名截断**: 自动截断超过50字符的提示文本，避免文件名过长。

3. **扩展名处理**: 自动将非 `.txt` 扩展名替换为 `.txt`。

4. **错误处理**: 文件写入失败时返回包含错误信息的完成消息。
