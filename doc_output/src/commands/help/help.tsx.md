# help.tsx — 显示帮助信息和可用命令

> **一句话总结**：渲染帮助界面，显示所有可用命令及其说明。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/help/help.tsx` |
| 文件类型 | TypeScript React (TSX) |
| 代码行数 | 10 行 |
| 主要职责 | 渲染HelpV2组件显示命令帮助信息 |

---

## 功能概述

该文件实现了 `/help` 命令，用于显示Claude Code的所有可用命令及其说明。它将渲染工作委托给 `HelpV2` 组件。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import { HelpV2 } from '../../components/HelpV2/HelpV2.js'
import type { LocalJSXCommandCall } from '../../types/command.js'
```

### 主要函数

#### call: LocalJSXCommandCall

- **类型**: `async function`
- **参数**:
  - `onDone`: 完成回调
  - `options`: 包含 `commands` 的命令列表
- **返回值**: `React.ReactNode`
- **用途**: 主入口函数

**执行逻辑**:

返回 `<HelpV2 commands={commands} onClose={onDone} />` 组件，传入所有命令列表和关闭回调。

---

## 设计要点

1. **组件化**: 将复杂的帮助界面逻辑委托给专门的 `HelpV2` 组件。

2. **类型安全**: 使用 `LocalJSXCommandCall` 类型确保符合命令调用接口。

3. **简洁实现**: 命令层仅负责传递参数，UI逻辑在组件中处理。

---

## 与其他文件的关系

**依赖**:
- `HelpV2` 组件 (`../../components/HelpV2/HelpV2.js`)
- `LocalJSXCommandCall` 类型 (`../../types/command.js`)

**被依赖**:
- `help/index.ts` - 导出为命令配置

---

## 注意事项

1. **HelpV2组件**: 这是一个V2版本的Help组件，说明帮助功能经过重构升级。

2. **命令列表**: 命令列表从 `options.commands` 传入，由命令系统自动提供。

3. **关闭回调**: 将 `onDone` 作为 `onClose` 传递给组件，用户关闭帮助时完成命令。
