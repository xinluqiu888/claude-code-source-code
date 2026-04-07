# index.ts — 文件列表命令配置

> **一句话总结**：定义 `/files` 命令的元数据配置，仅限内部用户使用。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/files/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 12 行 |
| 主要职责 | 导出files命令的配置对象，仅限特定用户类型 |

---

## 功能概述

该文件是文件列表命令的入口配置。这是一个内部调试命令，仅对 `USER_TYPE === 'ant'` 的用户启用。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '../../commands.js'
```

### 命令配置对象

- **类型**: `Command`
- **配置项**:
  - `type`: `'local'` - 本地命令类型（无UI）
  - `name`: `'files'` - 命令名称
  - `description`: `'List all files currently in context'` - 命令描述
  - `isEnabled`: `() => process.env.USER_TYPE === 'ant'` - 仅限ant用户
  - `supportsNonInteractive`: `true` - 支持非交互式会话
  - `load`: `() => import('./files.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `files` 命令配置对象

---

## 设计要点

1. **内部命令**: 通过 `isEnabled` 限制为仅特定用户类型可用。

2. **非交互式支持**: 支持在非交互式环境中使用，便于自动化脚本调试。

3. **本地命令**: 使用 `'local'` 类型，表示不渲染React UI。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **用户类型检查**: 硬编码检查 `USER_TYPE === 'ant'`，说明这是Anthropic内部调试命令。

2. **安全考虑**: 限制内部使用避免泄露上下文文件信息的敏感细节。
