# index.ts — 钩子命令配置

> **一句话总结**：定义 `/hooks` 命令的元数据配置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/hooks/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 11 行 |
| 主要职责 | 导出hooks命令的配置对象 |

---

## 功能概述

该文件是钩子命令的入口配置，用于查看和配置工具事件的钩子。

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
  - `name`: `'hooks'` - 命令名称
  - `description`: `'View hook configurations for tool events'` - 命令描述
  - `immediate`: `true` - 立即执行
  - `load`: `() => import('./hooks.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `hooks` 命令配置对象

---

## 设计要点

1. **即时执行**: `immediate: true` 表示命令立即执行，无需等待用户确认。

2. **交互式配置**: 使用JSX类型提供交互式钩子配置界面。

3. **工具事件**: 钩子系统与工具事件关联，允许自定义工具行为。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **钩子系统**: 这是一个高级功能，允许用户在工具执行前后插入自定义逻辑。

2. **即时命令**: 由于设置了immediate，用户输入后立即显示配置界面。
