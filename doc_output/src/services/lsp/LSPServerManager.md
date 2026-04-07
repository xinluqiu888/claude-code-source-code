# LSPServerManager.ts — LSP 服务器管理器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/lsp/LSPServerManager.ts`
- **所属模块**: LSP Service
- **功能类型**: 多服务器管理和请求路由

## 功能概述

该模块管理多个 LSP 服务器实例，基于文件扩展名路由请求。支持服务器延迟启动、文件同步（open/change/save/close）和生命周期管理。

## 核心内容详解

### LSPServerManager 接口

```typescript
{
  initialize(): Promise<void>                    // 加载配置并初始化
  shutdown(): Promise<void>                      // 关闭所有服务器
  getServerForFile(filePath): LSPServerInstance | undefined  // 获取文件对应服务器
  ensureServerStarted(filePath): Promise<LSPServerInstance | undefined>  // 确保服务器启动
  sendRequest<T>(filePath, method, params): Promise<T | undefined>  // 发送请求
  getAllServers(): Map<string, LSPServerInstance>  // 获取所有服务器
  openFile(filePath, content): Promise<void>     // 同步文件打开
  changeFile(filePath, content): Promise<void>   // 同步文件变更
  saveFile(filePath): Promise<void>              // 同步文件保存
  closeFile(filePath): Promise<void>             // 同步文件关闭
  isFileOpen(filePath): boolean                  // 检查文件是否已打开
}
```

### 主要函数

#### `createLSPServerManager(): LSPServerManager`
创建服务器管理器实例。

**内部状态：**
- `servers: Map<string, LSPServerInstance>` — 服务器实例映射
- `extensionMap: Map<string, string[]>` — 扩展名 → 服务器列表
- `openedFiles: Map<string, string>` — 文件 URI → 服务器名称

### 初始化

#### `initialize()`
加载配置并构建扩展名映射。

**流程：**
1. 调用 `getAllLspServers()` 获取配置
2. 遍历服务器配置：
   - 验证必需字段（command、extensionToLanguage）
   - 从 `extensionToLanguage` 提取文件扩展名
   - 构建 `extensionMap`（扩展名 → 服务器列表）
   - 创建 `LSPServerInstance`
   - 注册 `workspace/configuration` 处理器（返回 null）
3. 记录初始化结果

### 文件路由

#### `getServerForFile(filePath): LSPServerInstance | undefined`
根据文件路径获取对应服务器。

**策略：**
- 提取文件扩展名（小写）
- 查找 `extensionMap`
- 返回第一个匹配的服务器（后续可支持优先级）

#### `ensureServerStarted(filePath): Promise<LSPServerInstance | undefined>`
确保文件对应的服务器已启动。

**行为：**
- 服务器已运行 → 直接返回
- 服务器停止/错误 → 调用 `server.start()`
- 无匹配服务器 → 返回 undefined

### 请求发送

#### `sendRequest<T>(filePath, method, params): Promise<T | undefined>`
向文件对应的服务器发送请求。

**流程：**
1. 调用 `ensureServerStarted`
2. 调用 `server.sendRequest`
3. 错误时记录并传播

### 文件同步

#### `openFile(filePath, content)`
发送 `textDocument/didOpen` 通知。

**流程：**
1. 确保服务器启动
2. 转换路径为文件 URL
3. 检查文件是否已在同一服务器打开
4. 从 `extensionToLanguage` 获取 languageId
5. 发送 didOpen 通知
6. 记录到 `openedFiles`

#### `changeFile(filePath, content)`
发送 `textDocument/didChange` 通知。

**行为：**
- 服务器未运行/文件未打开 → 降级为 `openFile`
- 否则发送 didChange 通知

#### `saveFile(filePath)`
发送 `textDocument/didSave` 通知。

**用途：**
- 触发诊断刷新

#### `closeFile(filePath)`
发送 `textDocument/didClose` 通知。

**清理：**
- 从 `openedFiles` 移除
- 允许文件后续重新打开

#### `isFileOpen(filePath): boolean`
检查文件是否已在服务器打开。

### 关闭

#### `shutdown()`
关闭所有服务器并清理状态。

**流程：**
1. 收集运行中或错误状态的服务器
2. 并行调用 `server.stop()`
3. 清除所有状态（servers、extensionMap、openedFiles）
4. 收集并报告错误

## 设计要点

1. **延迟启动** — 服务器首次使用时启动，减少资源占用
2. **扩展名路由** — 简单高效的多服务器支持
3. **文件追踪** — 避免重复发送 didOpen
4. **错误隔离** — 单个服务器失败不影响其他
5. **闭包模式** — 状态封装，避免类继承

## 与其他文件的关系

- **被调用**: `manager.ts` 初始化和管理
- **依赖**: `config.ts` 加载配置
- **关联**: `LSPServerInstance.ts` 服务器实例

## 注意事项

- 多服务器支持同一扩展名时，取第一个注册
- `workspace/configuration` 请求返回 null（暂不支持配置）
- 文件关闭后可以从 `openedFiles` 移除，允许重新打开
- 变更前必须已打开（否则降级为打开）
