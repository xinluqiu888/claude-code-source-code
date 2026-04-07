# useMcpConnectivityStatus.tsx — MCP 连接状态通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useMcpConnectivityStatus.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `useMcpConnectivityStatus`
- **依赖**: React useEffect, notifications, MCP services

## 功能概述

本 Hook 监听 MCP（Model Context Protocol）客户端连接状态，在本地服务器失败、claude.ai 连接器失败或需要认证时显示相应通知。

## 核心内容详解

### 参数

```typescript
type Props = {
  mcpClients?: MCPServerConnection[]
}
```

### 客户端分类

**1. 本地失败客户端**
```typescript
const failedLocalClients = mcpClients.filter(
  client =>
    client.type === 'failed' &&
    client.config.type !== 'sse-ide' &&
    client.config.type !== 'ws-ide' &&
    client.config.type !== 'claudeai-proxy'
)
```

**2. Claude.ai 失败客户端**
```typescript
const failedClaudeAiClients = mcpClients.filter(
  client =>
    client.type === 'failed' &&
    client.config.type === 'claudeai-proxy' &&
    hasClaudeAiMcpEverConnected(client.name)
)
```

**3. 本地需认证**
```typescript
const needsAuthLocalServers = mcpClients.filter(
  client =>
    client.type === 'needs-auth' &&
    client.config.type !== 'claudeai-proxy'
)
```

**4. Claude.ai 需认证**
```typescript
const needsAuthClaudeAiServers = mcpClients.filter(
  client =>
    client.type === 'needs-auth' &&
    client.config.type === 'claudeai-proxy' &&
    hasClaudeAiMcpEverConnected(client.name)
)
```

### 通知格式

| 类型 | 消息 | 颜色 |
|------|------|------|
| 本地失败 | {n} MCP server(s) failed | error |
| Claude.ai 失败 | {n} claude.ai connector(s) unavailable | error |
| 本地需认证 | {n} MCP server(s) need auth | warning |
| Claude.ai 需认证 | {n} claude.ai connector(s) need auth | warning |

所有通知都包含 "· /mcp" 提示。

## 设计要点

### 1. 分类显示

本地服务器和 Claude.ai 连接器分开显示，因为后者通常表示服务中断而非配置问题。

### 2. 历史过滤

Claude.ai 相关通知只显示曾经成功连接过的连接器，避免组织配置的未使用连接器持续提醒。

### 3. IDE 客户端排除

IDE 类型的 MCP 客户端失败不显示，由专门的 IDE 状态处理器处理。

### 4. 批量显示

多个同类问题合并为一条通知，使用单复数适配。

## 与其他文件的关系

- **mcp/types.ts**: MCP 类型定义
- **claudeai.ts**: 提供 `hasClaudeAiMcpEverConnected`

## 注意事项

1. **区分失败原因**: 本地配置问题 vs 服务端问题
2. **历史感知**: 从未连接成功的 Claude.ai 连接器不提醒
3. **认证分离**: 本地和远程认证分开处理
4. **优先级适中**: 使用 'medium' 优先级，不打扰主要工作
