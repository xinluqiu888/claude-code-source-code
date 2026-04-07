# index.ts — MCP命令配置

> **一句话总结**：定义 `/mcp` 命令的元数据配置，支持启用/禁用服务器参数。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/mcp/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 13 行 |
| 主要职责 | 导出mcp命令的配置对象 |

---

## 功能概述

该文件是MCP（Model Context Protocol）服务器管理命令的入口配置。用户可以通过 `/mcp` 命令管理MCP服务器的启用和禁用状态。

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
  - `name`: `'mcp'` - 命令名称
  - `description`: `'Manage MCP servers'` - 命令描述
  - `immediate`: `true` - 立即执行命令
  - `argumentHint`: `'[enable|disable [server-name]]'` - 参数提示
  - `load`: `() => import('./mcp.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `mcp` 命令配置对象

---

## 设计要点

1. **立即执行**: `immediate: true` 表示命令会立即执行，不需要等待。

2. **参数支持**: 支持 `enable` 和 `disable` 子命令，可以指定服务器名称或操作所有服务器。

3. **交互式管理**: 基础命令会打开MCP设置界面进行交互式管理。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **立即执行**: 由于是立即执行命令，不需要额外确认。

2. **参数可选**: 不带参数时会打开交互式设置界面。
