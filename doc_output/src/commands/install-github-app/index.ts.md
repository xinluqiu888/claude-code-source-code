# index.ts — Install GitHub App命令配置

> **一句话总结**：定义 `/install-github-app` 命令的元数据配置，限制在claude-ai和console环境。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 14 行 |
| 主要职责 | 导出install-github-app命令的配置对象 |

---

## 功能概述

该文件是Install GitHub App命令的入口配置。用户可以通过 `/install-github-app` 命令为仓库设置Claude GitHub Actions。该命令仅在claude-ai和console环境中可用。

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
  - `name`: `'install-github-app'` - 命令名称
  - `description`: `'Set up Claude GitHub Actions for a repository'` - 命令描述
  - `availability`: `['claude-ai', 'console']` - 仅在claude-ai和console环境可用
  - `isEnabled`: `() => !isEnvTruthy(process.env.DISABLE_INSTALL_GITHUB_APP_COMMAND)` - 可通过环境变量禁用
  - `load`: `() => import('./install-github-app.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `installGitHubApp` 命令配置对象

---

## 设计要点

1. **环境限制**: `availability` 限制命令只在claude-ai和console环境可用。

2. **环境控制**: 通过 `DISABLE_INSTALL_GITHUB_APP_COMMAND` 环境变量可以禁用此命令。

3. **多步骤流程**: 命令包含多个步骤：检查GitHub CLI、选择仓库、配置API密钥、创建工作流等。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)
- `isEnvTruthy` (`../../utils/envUtils.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **环境专属**: 仅在claude-ai和console环境可用，其他环境看不到此命令。

2. **GitHub CLI依赖**: 命令执行需要GitHub CLI (gh) 已安装和认证。

3. **仓库权限**: 需要管理员权限才能设置GitHub Actions和密钥。
