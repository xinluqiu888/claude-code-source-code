# ReadMcpResourceTool.ts — MCP资源读取工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/ReadMcpResourceTool/ReadMcpResourceTool.ts`
- **作用**: 从MCP服务器读取特定资源的工具实现

## 功能概述

该工具允许从已连接的MCP服务器读取特定URI的资源。支持文本内容和二进制内容（blob）的处理，二进制内容会被持久化到磁盘。

## 核心内容详解

### 输入Schema

```typescript
z.object({
  server: z.string().describe('The MCP server name'),
  uri: z.string().describe('The resource URI to read'),
})
```

### 输出Schema

```typescript
z.object({
  contents: z.array(
    z.object({
      uri: z.string(),
      mimeType: z.string().optional(),
      text: z.string().optional(),
      blobSavedTo: z.string().optional(), // 二进制内容保存路径
    })
  ),
})
```

### 核心调用逻辑

1. **服务器查找**: 在mcpClients中查找指定名称的服务器
2. **连接检查**: 确保服务器已连接且支持resources功能
3. **资源读取**: 调用MCP协议`resources/read`方法
4. **内容处理**:
   - 文本内容：直接返回
   - 二进制内容：解码、写入磁盘，返回文件路径

### 二进制内容处理

```typescript
const persistId = `mcp-resource-${Date.now()}-${i}-${Math.random().toString(36).slice(2, 8)}`
const persisted = await persistBinaryContent(
  Buffer.from(c.blob, 'base64'),
  c.mimeType,
  persistId,
)
```

## 设计要点

1. **只读操作**: `isReadOnly() => true`
2. **并发安全**: `isConcurrencySafe() => true`
3. **延迟执行**: `shouldDefer: true`
4. **二进制处理**: 自动检测并持久化二进制内容
5. **错误处理**: 服务器不存在、未连接、不支持resources等错误场景

## 与其他文件的关系

- **MCP SDK**: 使用`ReadResourceResultSchema`进行类型验证
- **mcp/client.ts**: `ensureConnectedClient`确保客户端连接
- **mcpOutputStorage.ts**: `persistBinaryContent`处理二进制内容
- **UI.tsx**: 渲染函数定义

## 注意事项

- 工具返回的结果大小限制为100,000字符
- 二进制内容会被保存到临时目录，返回路径而非base64内容
