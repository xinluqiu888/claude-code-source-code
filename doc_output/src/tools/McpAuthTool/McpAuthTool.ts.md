# McpAuthTool.ts — MCP认证工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/McpAuthTool/McpAuthTool.ts`
- **作用**: 为需要认证的MCP服务器创建伪工具

## 功能概述

当MCP服务器已安装但未认证时，创建一个伪工具替代该服务器的真实工具。模型知道这个服务器存在，可以代表用户启动OAuth流程。

## 核心内容详解

### 输出类型

```typescript
export type McpAuthOutput = {
  status: 'auth_url' | 'unsupported' | 'error'
  message: string
  authUrl?: string
}
```

### 核心函数: createMcpAuthTool

```typescript
export function createMcpAuthTool(
  serverName: string,
  config: ScopedMcpServerConfig,
): Tool<InputSchema, McpAuthOutput>
```

### 工作流程

1. **检查配置类型**:
   - `claudeai-proxy`: 不支持，提示用户使用/mcp
   - 非sse/http传输: 不支持OAuth

2. **启动OAuth流程**:
   ```typescript
   const oauthPromise = performMCPOAuthFlow(
     serverName,
     sseOrHttpConfig,
     u => resolveAuthUrl?.(u),  // 捕获授权URL
     controller.signal,
     { skipBrowserOpen: true },
   )
   ```

3. **后台完成处理**:
   ```typescript
   void oauthPromise
     .then(async () => {
       clearMcpAuthCache()
       const result = await reconnectMcpServerImpl(serverName, config)
       // 用真实工具替换伪工具
       setAppState(prev => ({
         ...prev,
         mcp: {
           ...prev.mcp,
           tools: [...reject(prev.mcp.tools, t => t.name?.startsWith(prefix)),
                   ...result.tools],
         },
       }))
     })
   ```

4. **返回授权URL**:
   - 如果有authUrl: 提示用户打开浏览器授权
   - 如果静默完成: 告知用户认证成功

## 设计要点

1. **伪工具模式**: 在服务器未认证时代替真实工具
2. **前缀替换**: 使用`mcp__<server>__authenticate`命名
3. **自动刷新**: OAuth完成后自动替换为真实工具
4. **用户友好**: 提供清晰的授权指引

## 与其他文件的关系

- **services/mcp/auth.ts**: `performMCPOAuthFlow`
- **services/mcp/client.ts**: `reconnectMcpServerImpl`
- **services/mcp/mcpStringUtils.ts**: `buildMcpToolName`, `getMcpPrefix`

## 注意事项

- claude.ai连接器使用独立的认证流程
- OAuth令牌在进程内处理，永不暴露到shell
- 支持静默认证（已有缓存令牌）
