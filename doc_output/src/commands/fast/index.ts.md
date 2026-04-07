# index.ts — 快速模式命令配置

> **一句话总结**：定义 `/fast` 命令的元数据配置，支持动态描述和即时执行。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/fast/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 26 行 |
| 主要职责 | 导出fast命令的配置对象 |

---

## 功能概述

该文件是快速模式命令的入口配置。与静态配置不同，该配置使用getter动态计算某些属性（如描述文本），以反映当前系统状态。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '../../commands.js'
import {
  FAST_MODE_MODEL_DISPLAY,
  isFastModeEnabled,
} from '../../utils/fastMode.js'
import { shouldInferenceConfigCommandBeImmediate } from '../../utils/immediateCommand.js'
```

### 命令配置对象

- **类型**: `Command`
- **配置项**:
  - `type`: `'local-jsx'` - 本地JSX命令类型
  - `name`: `'fast'` - 命令名称
  - `description` (getter): 动态返回描述文本，格式为 `Toggle fast mode (${FAST_MODE_MODEL_DISPLAY} only)`
  - `availability`: `['claude-ai', 'console']` - 仅在claude-ai和console环境可用
  - `isEnabled`: `() => isFastModeEnabled()` - 动态检查功能是否启用
  - `isHidden` (getter): 如果快速模式未启用则隐藏命令
  - `argumentHint`: `'[on|off]'` - 参数提示
  - `immediate` (getter): 根据 `shouldInferenceConfigCommandBeImmediate()` 决定

### 对外导出

- **默认导出**: `fast` 命令配置对象

---

## 设计要点

1. **动态属性**: 使用getter使描述、可见性和即时性根据系统状态动态变化。

2. **功能开关**: `isEnabled` 和 `isHidden` 根据 `isFastModeEnabled()` 动态控制命令可用性。

3. **环境限制**: `availability` 限制命令只在特定环境可用。

4. **即时执行**: `immediate` 属性决定命令是否需要用户确认才执行。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)
- `FAST_MODE_MODEL_DISPLAY`, `isFastModeEnabled` (`../../utils/fastMode.js`)
- `shouldInferenceConfigCommandBeImmediate` (`../../utils/immediateCommand.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **Getter性能**: 动态属性每次访问都重新计算，确保反映最新状态。

2. **模型显示**: 描述中包含 `FAST_MODE_MODEL_DISPLAY`，向用户明确快速模式使用的模型。

3. **即时命令**: 当 `immediate` 为true时，命令输入后立即执行，不等待额外确认。
