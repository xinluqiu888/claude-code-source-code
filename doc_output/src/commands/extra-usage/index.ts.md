# index.ts — 额外使用量命令配置

> **一句话总结**：定义 `/extra-usage` 命令的配置，同时支持交互式和非交互式版本。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/extra-usage/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 31 行 |
| 主要职责 | 根据会话类型注册不同的extra-usage命令实现 |

---

## 功能概述

该文件根据当前会话是交互式还是非交互式，导出不同的命令配置。这是Claude Code中常见的模式，同一命令根据运行环境提供不同的实现。

---

## 核心内容详解

### 导入与依赖

```typescript
import { getIsNonInteractiveSession } from '../../bootstrap/state.js'
import type { Command } from '../../commands.js'
import { isOverageProvisioningAllowed } from '../../utils/auth.js'
import { isEnvTruthy } from '../../utils/envUtils.js'
```

### 辅助函数

#### isExtraUsageAllowed(): boolean

- **类型**: `function`
- **返回值**: `boolean`
- **用途**: 检查额外使用量功能是否被允许
- **逻辑**:
  - 如果环境变量 `DISABLE_EXTRA_USAGE_COMMAND` 为真，返回 `false`
  - 否则返回 `isOverageProvisioningAllowed()` 的结果

### 命令配置对象

#### extraUsage

- **类型**: `Command` (交互式版本)
- **配置**:
  - `type`: `'local-jsx'` - 本地JSX命令
  - `name`: `'extra-usage'`
  - `description`: `'Configure extra usage to keep working when limits are hit'`
  - `isEnabled`: `() => isExtraUsageAllowed() && !getIsNonInteractiveSession()` - 仅在交互式会话启用
  - `load`: `() => import('./extra-usage.js')` - 加载交互式实现

#### extraUsageNonInteractive

- **类型**: `Command` (非交互式版本)
- **配置**:
  - `type`: `'local'` - 本地命令（无UI）
  - `name`: `'extra-usage'`
  - `supportsNonInteractive`: `true` - 标记支持非交互式
  - `description`: 同上
  - `isEnabled`: `() => isExtraUsageAllowed() && getIsNonInteractiveSession()` - 仅在非交互式会话启用
  - `isHidden`: `() => !getIsNonInteractiveSession()` - 在交互式会话中隐藏
  - `load`: `() => import('./extra-usage-noninteractive.js')` - 加载非交互式实现

### 对外导出

- **具名导出**: `extraUsage`, `extraUsageNonInteractive`

---

## 设计要点

1. **环境适配**: 根据 `getIsNonInteractiveSession()` 动态决定启用哪个版本。

2. **互斥显示**: 通过 `isEnabled` 和 `isHidden` 确保两个版本不会同时显示在帮助中。

3. **功能开关**: 通过 `DISABLE_EXTRA_USAGE_COMMAND` 环境变量可以完全禁用此功能。

4. **权限检查**: 使用 `isOverageProvisioningAllowed()` 检查用户是否有权限使用此功能。

---

## 与其他文件的关系

**依赖**:
- `getIsNonInteractiveSession` (`../../bootstrap/state.js`)
- `isOverageProvisioningAllowed` (`../../utils/auth.js`)
- `isEnvTruthy` (`../../utils/envUtils.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **两个版本互斥**: 同一时刻只有一个版本会被启用，根据会话类型自动选择。

2. **命令名称相同**: 两个版本使用相同的命令名称，但类型和加载方式不同。

3. **非交互式版本标记**: `supportsNonInteractive: true` 告诉系统此命令可以在非交互式环境中运行。
