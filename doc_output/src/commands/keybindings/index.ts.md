# index.ts — 键盘快捷键命令配置

> **一句话总结**：定义 `/keybindings` 命令的元数据配置，仅在功能启用时可用。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/keybindings/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 13 行 |
| 主要职责 | 导出keybindings命令的配置对象 |

---

## 功能概述

该文件是键盘快捷键命令的入口配置。该命令仅在键盘快捷键自定义功能启用时可用。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '../../commands.js'
import { isKeybindingCustomizationEnabled } from '../../keybindings/loadUserBindings.js'
```

### 命令配置对象

- **类型**: `Command`
- **配置项**:
  - `name`: `'keybindings'` - 命令名称
  - `description`: `'Open or create your keybindings configuration file'` - 命令描述
  - `isEnabled`: `() => isKeybindingCustomizationEnabled()` - 功能启用时可用
  - `supportsNonInteractive`: `false` - 不支持非交互式
  - `type`: `'local'` - 本地命令类型
  - `load`: `() => import('./keybindings.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `keybindings` 命令配置对象

---

## 设计要点

1. **功能开关**: 通过 `isKeybindingCustomizationEnabled` 控制命令可用性。

2. **交互式必需**: 需要编辑器支持，不支持非交互式会话。

3. **预览功能**: 整体功能处于预览状态，可能不稳定。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)
- `isKeybindingCustomizationEnabled` (`../../keybindings/loadUserBindings.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **预览状态**: 键盘快捷键自定义功能处于预览状态，未来API可能变更。

2. **非交互式限制**: 在CI/CD等环境中不可用。
