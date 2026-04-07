# output-style.tsx — 已弃用的输出样式命令

> **一句话总结**：显示弃用提示，引导用户使用 `/config` 命令。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/output-style/output-style.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 7 行 |
| 主要职责 | 显示命令弃用提示信息 |

---

## 功能概述

该文件实现了已弃用的 `/output-style` 命令。当用户执行此命令时，会显示弃用信息，引导用户使用 `/config` 命令或在设置文件中修改输出样式。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { LocalJSXCommandOnDone } from '../../types/command.js'
```

### 主要函数

#### call(onDone): Promise<undefined>

- **类型**: `async function`
- **参数**:
  - `onDone`: `LocalJSXCommandOnDone` - 完成回调
- **返回值**: `Promise<undefined>`
- **用途**: 命令入口函数

**执行逻辑**:

1. 调用 `onDone` 返回弃用提示信息
2. 使用 `display: 'system'` 以系统消息样式显示

**提示内容**: "`/output-style` has been deprecated. Use `/config` to change your output style, or set it in your settings file. Changes take effect on the next session."

---

## 设计要点

1. **简单实现**: 仅返回一条弃用提示，不执行任何实际操作。

2. **系统消息样式**: 使用 `display: 'system'` 使消息以系统提示样式显示。

---

## 与其他文件的关系

**依赖**:
- `LocalJSXCommandOnDone` 类型 (`../../types/command.js`)

**被依赖**:
- `output-style/index.ts` - 导出为命令配置

---

## 注意事项

1. **无实际操作**: 此命令仅显示弃用提示，不会更改任何设置。

2. **配置替代**: 用户需要使用 `/config` 命令来更改输出样式。

3. **会话生效**: 输出样式更改需要在新会话中才能生效。
