# index.ts — 导出命令配置

> **一句话总结**：定义 `/export [filename]` 命令的元数据配置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/export/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 11 行 |
| 主要职责 | 导出export命令的配置对象 |

---

## 功能概述

该文件是对话导出命令的入口配置，定义了命令的元数据。用户可通过 `/export` 或 `/export filename.txt` 将当前对话导出为文本文件。

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
  - `name`: `'export'` - 命令名称
  - `description`: `'Export the current conversation to a file or clipboard'` - 命令描述
  - `argumentHint`: `'[filename]'` - 参数提示（可选文件名）
  - `load`: `() => import('./export.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `exportCommand` 命令配置对象

---

## 设计要点

1. **可选参数**: `argumentHint` 提示用户可以提供文件名参数，但非必需。

2. **懒加载**: 动态导入减少初始加载时间。

3. **JSX命令类型**: 使用 `'local-jsx'` 类型表示该命令会渲染React组件。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **参数可选性**: 用户可以不提供文件名，此时会显示导出对话框。
