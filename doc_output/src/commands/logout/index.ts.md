# index.ts — 登出命令配置

> **一句话总结**：定义 `/logout` 命令的元数据配置，支持环境变量控制启用状态。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/logout/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 11 行 |
| 主要职责 | 导出logout命令的配置对象 |

---

## 功能概述

该文件是登出命令的入口配置。用户可以通过 `/logout` 命令登出Anthropic账户，清除所有认证相关的缓存和凭证。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '../../commands.js'
import { isEnvTruthy } from '../../utils/envUtils.js'
```

### 命令配置对象

- **类型**: `Command`
- **配置项**:
  - `type`: `'local-jsx'` - 本地JSX命令类型
  - `name`: `'logout'` - 命令名称
  - `description`: `'Sign out from your Anthropic account'` - 命令描述
  - `isEnabled`: `() => !isEnvTruthy(process.env.DISABLE_LOGOUT_COMMAND)` - 可通过环境变量禁用
  - `load`: `() => import('./logout.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `logout` 命令配置对象

---

## 设计要点

1. **环境控制**: 通过 `DISABLE_LOGOUT_COMMAND` 环境变量可以禁用登出命令。

2. **交互式命令**: 使用JSX类型，提供交互式界面。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)
- `isEnvTruthy` (`../../utils/envUtils.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **环境变量**: 设置 `DISABLE_LOGOUT_COMMAND=1` 可禁用此命令。

2. **完全登出**: 登出操作会清除安全存储中的所有数据。
