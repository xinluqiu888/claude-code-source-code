# index.ts — 登录命令配置

> **一句话总结**：定义 `/login` 命令的元数据配置，支持账号切换和登录状态检测。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/login/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 14 行 |
| 主要职责 | 导出login命令的配置对象 |

---

## 功能概述

该文件是登录命令的入口配置。根据用户当前的认证状态，命令描述会动态变化：已登录用户显示"Switch Anthropic accounts"，未登录用户显示"Sign in with your Anthropic account"。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '../../commands.js'
import { hasAnthropicApiKeyAuth } from '../../utils/auth.js'
import { isEnvTruthy } from '../../utils/envUtils.js'
```

### 命令配置对象

- **类型**: `Command`
- **配置项**:
  - `type`: `'local-jsx'` - 本地JSX命令类型
  - `name`: `'login'` - 命令名称
  - `description`: 动态描述，根据 `hasAnthropicApiKeyAuth()` 返回值变化
  - `isEnabled`: `() => !isEnvTruthy(process.env.DISABLE_LOGIN_COMMAND)` - 可通过环境变量禁用
  - `load`: `() => import('./login.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: 返回登录命令配置对象的函数

---

## 设计要点

1. **动态描述**: 根据认证状态显示不同的命令描述，提升用户体验。

2. **环境控制**: 通过 `DISABLE_LOGIN_COMMAND` 环境变量可以禁用登录命令。

3. **懒加载**: 使用动态导入延迟加载实际执行逻辑。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)
- `hasAnthropicApiKeyAuth` (`../../utils/auth.js`)
- `isEnvTruthy` (`../../utils/envUtils.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **环境变量**: 设置 `DISABLE_LOGIN_COMMAND=1` 可禁用此命令。

2. **描述变化**: 命令描述会根据用户登录状态自动切换语言。
