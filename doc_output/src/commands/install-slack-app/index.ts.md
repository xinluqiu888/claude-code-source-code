# index.ts — 安装Slack应用命令配置

> **一句话总结**：定义 `/install-slack-app` 命令的元数据配置，限制在claude-ai环境。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-slack-app/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 12 行 |
| 主要职责 | 导出install-slack-app命令的配置对象 |

---

## 功能概述

该文件是安装Slack应用命令的入口配置。该命令仅在claude-ai环境中可用。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '../../commands.js'
```

### 命令配置对象

- **类型**: `Command`
- **配置项**:
  - `type`: `'local'` - 本地命令类型
  - `name`: `'install-slack-app'` - 命令名称
  - `description`: `'Install the Claude Slack app'` - 命令描述
  - `availability`: `['claude-ai']` - 仅在claude-ai环境可用
  - `supportsNonInteractive`: `false` - 不支持非交互式会话
  - `load`: `() => import('./install-slack-app.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `installSlackApp` 命令配置对象

---

## 设计要点

1. **环境限制**: `availability: ['claude-ai']` 限制命令只在claude-ai环境可用。

2. **交互式必需**: `supportsNonInteractive: false` 表示必须有浏览器才能使用。

3. **本地命令**: 不使用JSX，直接执行浏览器打开操作。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **环境专属**: 仅在claude-ai环境可用，其他环境看不到此命令。

2. **需要浏览器**: 不支持非交互式会话，需要能够打开浏览器。
