# index.ts — 帮助命令配置

> **一句话总结**：定义 `/help` 命令的元数据配置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/help/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 10 行 |
| 主要职责 | 导出help命令的配置对象 |

---

## 功能概述

该文件是帮助命令的入口配置，用户可以通过 `/help` 查看所有可用命令。

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
  - `name`: `'help'` - 命令名称
  - `description`: `'Show help and available commands'` - 命令描述
  - `load`: `() => import('./help.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `help` 命令配置对象

---

## 设计要点

1. **基本命令**: help是最基础的命令之一，不设置isEnabled表示总是可用。

2. **交互式UI**: 使用JSX类型，提供交互式的帮助界面。

3. **无参数**: 简单的无参数命令，直接显示帮助信息。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **自动提供commands**: 命令列表由系统从options.commands自动提供。

2. **HelpV2组件**: 实际的帮助UI由HelpV2组件渲染。
