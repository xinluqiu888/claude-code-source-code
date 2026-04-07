# MCPConnectionManager.tsx — MCP 连接管理器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/mcp/MCPConnectionManager.tsx`
- **所属模块**: MCP Service
- **功能类型**: React 上下文提供器

## 功能概述

React 组件，提供 MCP 连接管理功能给子组件。封装 `useManageMCPConnections` hook。

## 核心内容详解

### Context

```typescript
interface MCPConnectionContextValue {
  reconnectMcpServer: (serverName: string) => Promise<{
    client: MCPServerConnection
    tools: Tool[]
    commands: Command[]
    resources?: ServerResource[]
  }>
  toggleMcpServer: (serverName: string) => Promise<void>
}
```

### Hooks

#### `useMcpReconnect()`
获取重新连接服务器函数。

#### `useMcpToggleEnabled()`
获取切换服务器启用状态函数。

### 组件

#### `MCPConnectionManager`
提供器组件，接收：
- `children` — 子组件
- `dynamicMcpConfig` — 动态 MCP 配置
- `isStrictMcpConfig` — 是否严格配置模式

## 设计要点

1. **React Context** — 跨组件共享 MCP 功能
2. **Hook 封装** — 逻辑在 `useManageMCPConnections`
3. **类型安全** — TypeScript 完整类型

## 与其他文件的关系

- **依赖**: `useManageMCPConnections.ts`
- **使用**: React 组件树

## 注意事项

- 必须在 `MCPConnectionManager` 内使用 hooks
- 编译后代码包含 React Compiler 运行时
