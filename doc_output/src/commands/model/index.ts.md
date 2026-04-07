# index.ts — Model命令配置

> **一句话总结**：定义 `/model` 命令的元数据配置，动态显示当前模型名称。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/model/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 17 行 |
| 主要职责 | 导出model命令的配置对象 |

---

## 功能概述

该文件是Model命令的入口配置。用户可以通过 `/model` 命令查看和设置当前使用的AI模型。命令描述会动态显示当前模型名称。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '../../commands.js'
import { shouldInferenceConfigCommandBeImmediate } from '../../utils/immediateCommand.js'
import { getMainLoopModel, renderModelName } from '../../utils/model/model.js'
```

### 命令配置对象

- **类型**: `Command`
- **配置项**:
  - `type`: `'local-jsx'` - 本地JSX命令类型
  - `name`: `'model'` - 命令名称
  - `description`: 动态描述，显示当前模型名称
  - `argumentHint`: `'[model]'` - 参数提示
  - `immediate`: 根据 `shouldInferenceConfigCommandBeImmediate()` 动态确定
  - `load`: `() => import('./model.js')` - 懒加载执行函数

### 动态属性

```typescript
get description() {
  return `Set the AI model for Claude Code (currently ${renderModelName(getMainLoopModel())})`
}
get immediate() {
  return shouldInferenceConfigCommandBeImmediate()
}
```

### 对外导出

- **默认导出**: model命令配置对象

---

## 设计要点

1. **动态描述**: 命令描述实时显示当前使用的模型名称，提升用户体验。

2. **立即执行控制**: `immediate` 属性根据配置动态确定，支持灵活的控制策略。

3. **参数支持**: 支持通过参数直接设置模型，如 `/model claude-4-6`。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)
- `shouldInferenceConfigCommandBeImmediate` (`../../utils/immediateCommand.js`)
- `getMainLoopModel`, `renderModelName` (`../../utils/model/model.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **模型渲染**: `renderModelName` 函数将内部模型标识转换为可读的显示名称。

2. **立即执行**: 当 `immediate` 为true时，命令会立即执行而不需要额外确认。
