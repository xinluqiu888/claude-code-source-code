# mcp.tsx — MCP服务器管理

> **一句话总结**：实现MCP服务器的启用/禁用管理和交互式设置界面。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/mcp/mcp.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 84 行 |
| 主要职责 | 实现MCP服务器的交互式管理和快速启用/禁用功能 |

---

## 功能概述

该文件实现了 `/mcp` 命令的功能。支持通过参数快速启用/禁用MCP服务器，也可以不带参数打开交互式设置界面。对于ant用户，基础命令会重定向到插件管理界面。

---

## 核心内容详解

### 导入与依赖

```typescript
import React, { useEffect, useRef } from 'react'
import { MCPSettings } from '../../components/mcp/index.js'
import { MCPReconnect } from '../../components/mcp/MCPReconnect.js'
import { useMcpToggleEnabled } from '../../services/mcp/MCPConnectionManager.js'
import { useAppState } from '../../state/AppState.js'
import type { LocalJSXCommandOnDone } from '../../types/command.js'
import { PluginSettings } from '../plugin/PluginSettings.js'
```

### 主要组件

#### MCPToggle(props): null

- **类型**: React函数组件
- **参数**:
  - `action`: `'enable' | 'disable'` - 操作类型
  - `target`: `string` - 目标服务器名称或'all'
  - `onComplete`: `(result: string) => void` - 完成回调
- **用途**: 内部组件，用于切换MCP服务器的启用状态

**实现逻辑**:

1. 使用 `useAppState` 获取所有MCP客户端
2. 过滤掉名称为 "ide" 的客户端
3. 根据操作类型和过滤条件确定要切换的服务器列表
4. 遍历并切换每个服务器的启用状态
5. 调用 `onComplete` 返回操作结果

#### call(onDone, _context, args?): Promise<React.ReactNode>

- **类型**: `async function`
- **返回值**: `Promise<React.ReactNode>`
- **用途**: 命令入口函数

**参数处理**:

- `no-redirect`: 绕过重定向，直接显示MCP设置
- `reconnect <server-name>`: 重新连接指定服务器
- `enable [server-name]`: 启用指定服务器或所有服务器
- `disable [server-name]`: 禁用指定服务器或所有服务器
- 无参数: 对于ant用户重定向到插件管理界面，其他用户显示MCP设置

---

## 设计要点

1. **Context Hack**: 组件内使用 `useContext` 获取 `toggleMcpServer`，理想情况下所有MCP状态和函数应该在全局状态中。

2. **IDE客户端过滤**: 名称为 "ide" 的MCP客户端被排除在切换操作之外。

3. **Ant用户重定向**: ant用户执行 `/mcp` 会被重定向到 `/plugins` 的已安装标签页。

---

## 与其他文件的关系

**依赖**:
- `MCPSettings` (`../../components/mcp/index.js`)
- `MCPReconnect` (`../../components/mcp/MCPReconnect.js`)
- `useMcpToggleEnabled` (`../../services/mcp/MCPConnectionManager.js`)
- `useAppState` (`../../state/AppState.js`)
- `PluginSettings` (`../plugin/PluginSettings.js`)

**被依赖**:
- `mcp/index.ts` - 导出为命令配置

---

## 注意事项

1. **TODO**: 注释中提到这是一个hack，理想情况下应该将所有MCP状态和函数放到全局状态中。

2. **IDE保护**: "ide" MCP服务器被保护，不会被批量启用/禁用操作影响。

3. **Reconnnect功能**: 支持通过 `/mcp reconnect <server-name>` 重新连接特定服务器。
