# index.ts — 退出命令配置

> **一句话总结**：定义 `/exit` 和 `/quit` 命令的元数据配置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/exit/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 12 行 |
| 主要职责 | 导出exit命令的配置对象 |

---

## 功能概述

该文件是退出命令的入口配置，定义了命令的元数据，包括命令名称、别名、描述和加载方式。使用Command类型进行类型检查，确保配置符合系统要求。

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
  - `name`: `'exit'` - 命令主名称
  - `aliases`: `['quit']` - 命令别名数组
  - `description`: `'Exit the REPL'` - 命令描述
  - `immediate`: `true` - 立即执行，不等待用户确认
  - `load`: `() => import('./exit.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `exit` 命令配置对象

---

## 设计要点

1. **懒加载**: 使用动态导入 `import('./exit.js')` 实现代码分割，只在命令执行时加载。

2. **类型安全**: 使用 `satisfies Command` 确保配置符合Command接口。

3. **多别名支持**: 同时支持 `/exit` 和 `/quit` 两个命令。

4. **immediate标志**: 设置为true表示命令立即执行，无需二次确认。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`) - 命令配置接口定义

**被依赖**:
- 命令注册系统 - 将该命令注册到REPL命令表中

---

## 注意事项

1. **模块路径**: 懒加载路径 `'./exit.js'` 是编译后的JS路径，源文件是 `.tsx`。

2. **immediate执行**: 由于设置了 `immediate: true`，用户输入 `/exit` 后立即执行，不会提示确认。
