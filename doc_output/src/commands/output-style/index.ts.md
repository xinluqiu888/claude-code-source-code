# index.ts — Output Style命令配置

> **一句话总结**：定义已弃用的 `/output-style` 命令配置，引导用户使用 `/config`。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/output-style/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 12 行 |
| 主要职责 | 导出已弃用的output-style命令配置 |

---

## 功能概述

该文件是Output Style命令的入口配置。此命令已被弃用，用户使用时会收到提示，引导他们使用 `/config` 命令或设置文件来更改输出样式。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '../../commands.js'
```

### 命令配置对象

- **类型**: `Command`
- **配置项**:
  - `type`: `'local-jsx'` - 本地JSX命令类型
  - `name`: `'output-style'` - 命令名称
  - `description`: `'Deprecated: use /config to change output style'` - 弃用说明
  - `isHidden`: `true` - 隐藏命令，不在帮助中显示
  - `load`: `() => import('./output-style.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `outputStyle` 命令配置对象

---

## 设计要点

1. **隐藏命令**: `isHidden: true` 使命令不在帮助列表中显示。

2. **弃用提示**: 命令执行时会显示弃用信息，引导用户使用 `/config`。

3. **向后兼容**: 保留命令以支持可能仍在使用旧命令的脚本或用户。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **不再维护**: 此命令已弃用，不会接受新功能。

2. **配置替代**: 用户应使用 `/config` 命令或修改设置文件来更改输出样式。

3. **下次生效**: 输出样式更改需要在新会话中才能生效。
