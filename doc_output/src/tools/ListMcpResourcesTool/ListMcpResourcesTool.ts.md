# ListMcpResourcesTool.ts — MCP资源列表工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/ListMcpResourcesTool/ListMcpResourcesTool.ts`
- **工具名称**: `ListMcpResourcesTool`
- **用户面名称**: `listMcpResources`

## 功能概述

列出配置的MCP（Model Context Protocol）服务器中可用的资源。支持按服务器名称筛选或列出所有服务器的资源。

## 核心内容详解

### 输入模式

```typescript
const inputSchema = lazySchema(() =>
  z.object({
    server: z.string().optional()
      .describe('Optional server name to filter resources by'),
  })
)
```

### 输出模式

```typescript
const outputSchema = lazySchema(() =>
  z.array(
    z.object({
      uri: z.string(),
      name: z.string(),
      mimeType: z.string().optional(),
      description: z.string().optional(),
      server: z.string(),
    })
  )
)
```

### 核心逻辑

```typescript
async call(input, { options: { mcpClients } }) {
  const { server: targetServer } = input
  
  // 筛选目标服务器
  const clientsToProcess = targetServer
    ? mcpClients.filter(client => client.name === targetServer)
    : mcpClients
  
  // 并行获取所有服务器的资源
  const results = await Promise.all(
    clientsToProcess.map(async client => {
      if (client.type !== 'connected') return []
      try {
        const fresh = await ensureConnectedClient(client)
        return await fetchResourcesForClient(fresh)
      } catch (error) {
        logMCPError(client.name, errorMessage(error))
        return []
      }
    })
  )
  
  return { data: results.flat() }
}
```

### 缓存机制

- `fetchResourcesForClient`使用LRU缓存（按服务器名称）
- 在启动时预取，保持缓存热度
- 在onclose和resources/list_changed通知时使缓存失效

## 设计要点

1. **只读操作**: `isReadOnly: true`
2. **并发安全**: `isConcurrencySafe: true`
3. **延迟执行**: `shouldDefer: true`
4. **容错处理**: 单个服务器失败不影响整体结果
5. **结果大小限制**: `maxResultSizeChars: 100_000`

## 与其他文件的关系

- **prompt.ts**: 工具描述和提示
- **UI.tsx**: 渲染函数
- **services/mcp/client.ts**: MCP客户端管理

## 注意事项

- MCP服务器可能没有资源但仍提供工具
- 返回的结果包含server字段指示来源
