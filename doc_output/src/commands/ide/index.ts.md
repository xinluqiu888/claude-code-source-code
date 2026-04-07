# index.ts — IDE命令配置

> **一句话总结**：定义 `/ide` 命令的元数据配置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/ide/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 11 行 |
| 主要职责 | 导出ide命令的配置对象 |

---

## 功能概述

该文件是IDE命令的入口配置，用于管理Claude Code与IDE的集成。

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
  - `name`: `'ide'` - 命令名称
  - `description`: `'Manage IDE integrations and show status'` - 命令描述
  - `argumentHint`: `'[open]'` - 可选open参数
  - `load`: `() => import('./ide.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `ide` 命令配置对象

---

## 设计要点

1. **可选参数**: `open` 参数用于在IDE中打开当前目录。

2. **交互式管理**: 使用JSX类型提供交互式IDE选择和配置界面。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **IDE检测**: 需要系统支持IDE检测，可能在某些环境中无法工作。

2. **open参数**: 如果提供了open参数但未选中IDE，可能不会有任何效果。
