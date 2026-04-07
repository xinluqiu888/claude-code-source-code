# index.ts — Memory命令配置

> **一句话总结**：定义 `/memory` 命令的元数据配置，用于编辑Claude记忆文件。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/memory/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 11 行 |
| 主要职责 | 导出memory命令的配置对象 |

---

## 功能概述

该文件是Memory命令的入口配置。用户可以通过 `/memory` 命令编辑Claude的记忆文件，这些文件用于存储用户偏好和上下文信息。

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
  - `name`: `'memory'` - 命令名称
  - `description`: `'Edit Claude memory files'` - 命令描述
  - `load`: `() => import('./memory.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `memory` 命令配置对象

---

## 设计要点

1. **交互式编辑**: 使用JSX类型提供交互式记忆文件选择和编辑界面。

2. **懒加载**: 使用动态导入延迟加载实际执行逻辑。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **记忆文件**: 记忆文件存储在Claude配置目录中，用于持久化用户偏好。

2. **文件创建**: 如果记忆文件不存在，会自动创建空文件。
