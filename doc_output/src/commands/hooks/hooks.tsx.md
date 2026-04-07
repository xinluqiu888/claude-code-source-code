# hooks.tsx — 显示和配置工具事件钩子

> **一句话总结**：显示工具事件钩子的配置界面，支持查看和修改钩子设置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/hooks/hooks.tsx` |
| 文件类型 | TypeScript React (TSX) |
| 代码行数 | 12 行 |
| 主要职责 | 渲染钩子配置菜单组件 |

---

## 功能概述

该文件实现了 `/hooks` 命令，用于查看和管理工具事件的钩子配置。钩子允许用户在特定工具事件发生时执行自定义操作。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import { HooksConfigMenu } from '../../components/hooks/HooksConfigMenu.js'
import { logEvent } from '../../services/analytics/index.js'
import { getTools } from '../../tools.js'
import type { LocalJSXCommandCall } from '../../types/command.js'
```

### 主要函数

#### call: LocalJSXCommandCall

- **类型**: `async function`
- **参数**:
  - `onDone`: 完成回调
  - `context`: 命令上下文
- **返回值**: `React.ReactNode`
- **用途**: 主入口函数

**执行流程**:

1. 记录分析事件：`logEvent('tengu_hooks_command', {})`
2. 获取应用状态：`context.getAppState()`
3. 提取权限上下文：`appState.toolPermissionContext`
4. 获取工具列表：`getTools(permissionContext)`
5. 提取工具名称：`tools.map(tool => tool.name)`
6. 返回 `<HooksConfigMenu toolNames={toolNames} onExit={onDone} />`

---

## 设计要点

1. **分析追踪**: 记录hooks命令的使用情况用于分析。

2. **动态工具列表**: 从工具系统获取当前可用的工具列表。

3. **权限上下文**: 使用toolPermissionContext确保只显示用户有权限的工具。

4. **组件化**: 复杂的配置UI委托给HooksConfigMenu组件。

---

## 与其他文件的关系

**依赖**:
- `HooksConfigMenu` 组件 (`../../components/hooks/HooksConfigMenu.js`)
- `logEvent` (`../../services/analytics/index.js`)
- `getTools` (`../../tools.js`)

**被依赖**:
- `hooks/index.ts` - 导出为命令配置

---

## 注意事项

1. **工具钩子**: 钩子系统允许在工具执行前后插入自定义逻辑。

2. **权限过滤**: 通过toolPermissionContext过滤工具列表，只显示有权限的工具。

3. **即时执行**: immediate: true表示命令立即执行，无需二次确认。
