# LSPClient.ts — LSP 客户端实现

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/lsp/LSPClient.ts`
- **所属模块**: LSP Service
- **功能类型**: 语言服务器协议客户端

## 功能概述

该模块实现 LSP（Language Server Protocol）客户端，使用 `vscode-jsonrpc` 库通过 stdio 与 LSP 服务器进程通信。管理连接生命周期、请求/通知发送和错误处理。

## 核心内容详解

### LSPClient 接口

```typescript
{
  capabilities: ServerCapabilities | undefined  // 服务器能力
  isInitialized: boolean                          // 是否已初始化
  start(command, args, options): Promise<void>   // 启动服务器进程
  initialize(params): Promise<InitializeResult>  // 初始化协议
  sendRequest<TResult>(method, params): Promise<TResult>  // 发送请求
  sendNotification(method, params): Promise<void>         // 发送通知
  onNotification(method, handler): void          // 注册通知处理器
  onRequest(method, handler): void               // 注册请求处理器
  stop(): Promise<void>                          // 停止客户端
}
```

### 主要函数

#### `createLSPClient(serverName, onCrash?): LSPClient`
创建 LSP 客户端实例。

**参数：**
- `serverName` — 服务器名称（用于日志）
- `onCrash` — 服务器崩溃时的回调

**内部状态：**
- `process` — 子进程实例
- `connection` — JSON-RPC 连接
- `capabilities` — 服务器能力
- `isInitialized` — 初始化状态
- `pendingHandlers` — 连接就绪前注册的通知处理器队列
- `pendingRequestHandlers` — 连接就绪前注册的请求处理器队列

### 生命周期

#### `start(command, args, options)`
启动 LSP 服务器进程。

**流程：**
1. 使用 `spawn` 创建子进程（stdio: pipe）
2. 等待进程成功启动（监听 `spawn` 和 `error` 事件）
3. 设置 stderr 数据处理器（记录服务器日志）
4. 设置进程错误和退出处理器
5. 创建 JSON-RPC 连接（StreamMessageReader/Writer）
6. 注册连接错误和关闭处理器
7. 启动连接监听
8. 启用协议追踪（调试）
9. 应用待处理的处理器

**关键安全措施：**
- 等待 `spawn` 事件确认进程启动成功
- 处理 `ENOENT` 等启动错误
- stdin 错误处理防止未处理的 Promise 拒绝

#### `initialize(params)`
执行 LSP 初始化握手。

**流程：**
1. 发送 `initialize` 请求
2. 保存服务器能力
3. 发送 `initialized` 通知
4. 标记为已初始化

#### `stop()`
优雅关闭 LSP 客户端。

**流程：**
1. 标记 `isStopping = true`（防止错误处理器记录误报）
2. 发送 `shutdown` 请求
3. 发送 `exit` 通知
4. 释放连接资源
5. 移除进程事件监听器
6. 终止进程
7. 重置状态

### 错误处理

**启动错误：**
- 设置 `startFailed` 和 `startError`
- 后续操作抛出包含错误信息的异常

**运行时错误：**
- 连接错误：记录并设置失败状态
- 进程退出（非零）：调用 `onCrash` 回调
- 请求/通知错误：记录并传播

**优雅关闭：**
- `isStopping` 标志防止误报
- 无论关闭成功与否都清理资源

### 延迟初始化支持

处理器可在连接就绪前注册，自动加入队列：
- `onNotification` → `pendingHandlers`
- `onRequest` → `pendingRequestHandlers`

连接建立后自动应用队列中的处理器。

## 设计要点

1. **闭包模式** — 使用工厂函数而非类，状态封装在闭包中
2. **防御性编程** — 多重错误边界防止未处理的 Promise 拒绝
3. **优雅降级** — 通知失败不中断流程（fire-and-forget）
4. **资源清理** — 确保进程、连接、监听器正确释放
5. **调试支持** — 协议追踪和详细日志

## 与其他文件的关系

- **被调用**: `LSPServerInstance.ts` 创建服务器实例
- **依赖**: `vscode-jsonrpc/node.js` JSON-RPC 实现
- **关联**: `subprocessEnv.ts` 子进程环境变量

## 注意事项

- `windowsHide: true` 防止 Windows 上显示控制台窗口
- 协议追踪启用失败被捕获但不中断流程
- 崩溃检测依赖非零退出码
- 处理器队列在连接建立后自动清空
