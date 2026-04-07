# createDirectConnectSession.ts — 直连会话创建

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/server/createDirectConnectSession.ts`
- **类型**: TypeScript 模块
- **作用**: 在直连服务器上创建新会话

## 功能概述

本模块提供函数用于向直连服务器发送会话创建请求，处理HTTP通信、响应验证和错误处理。它是建立直连会话的第一步。

## 核心内容详解

### 错误类型

```typescript
export class DirectConnectError extends Error {
  constructor(message: string) {
    super(message)
    this.name = 'DirectConnectError'
  }
}
```

专用的错误类型用于区分直连连接失败与其他错误。

### 会话创建函数

```typescript
export async function createDirectConnectSession({
  serverUrl,
  authToken,
  cwd,
  dangerouslySkipPermissions,
}: {
  serverUrl: string
  authToken?: string
  cwd: string
  dangerouslySkipPermissions?: boolean
}): Promise<{
  config: DirectConnectConfig
  workDir?: string
}>
```

### 功能流程

1. **准备请求头**:
   - `content-type: application/json`
   - `authorization: Bearer ${authToken}` (如果提供)

2. **发送POST请求**:
   - URL: `${serverUrl}/sessions`
   - Body: `{ cwd, dangerously_skip_permissions?: boolean }`

3. **错误处理**:
   - 网络错误: "Failed to connect to server at ${serverUrl}"
   - HTTP错误: "Failed to create session: ${status} ${statusText}"

4. **响应验证**:
   - 使用`connectResponseSchema`验证响应格式
   - 无效响应: "Invalid session response: ${error.message}"

5. **返回配置**:
   ```typescript
   return {
     config: {
       serverUrl,
       sessionId: data.session_id,
       wsUrl: data.ws_url,
       authToken,
     },
     workDir: data.work_dir,
   }
   ```

### 响应Schema

通过`connectResponseSchema()`验证（来自types.ts）：
```typescript
{
  session_id: string
  ws_url: string
  work_dir?: string
}
```

## 设计要点

1. **专用错误类**: 便于调用方区分处理
2. **Schema验证**: 使用Zod确保响应格式正确
3. **可选authToken**: 支持认证和非认证模式
4. **危险标志**: `dangerouslySkipPermissions`用于自动化场景

## 与其他文件的关系

- **被 directConnectManager.ts 使用**: 创建会话后建立WebSocket连接
- **使用 types.ts**: 导入`connectResponseSchema`和`DirectConnectConfig`
- **使用 slowOperations.ts**: `jsonStringify`用于请求体序列化
- **被 REPL.tsx 使用**: 通过`useDirectConnect` hook使用

## 注意事项

1. **错误区分**: 网络错误、HTTP错误、验证错误分别处理
2. **AuthToken可选**: 服务器可能不需要认证
3. **workDir可选**: 服务器可能不返回工作目录
4. **同步序列化**: 使用`jsonStringify`而非JSON.stringify以支持BigInt
