# manager.ts — LSP 管理器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/lsp/manager.ts`
- **所属模块**: LSP Service
- **功能类型**: 全局管理器生命周期

## 功能概述

该模块管理全局 LSP 服务器管理器单例的生命周期，包括初始化、状态跟踪、重新初始化和关闭。

## 核心内容详解

### 状态管理

```typescript
type InitializationState = 'not-started' | 'pending' | 'success' | 'failed'

let lspManagerInstance: LSPServerManager | undefined
let initializationState: InitializationState = 'not-started'
let initializationError: Error | undefined
let initializationGeneration: number = 0
let initializationPromise: Promise<void> | undefined
```

### 主要函数

#### `initializeLspServerManager(): void`
初始化 LSP 服务器管理器。

**流程：**
1. 检查 bare 模式（`--bare` / `SIMPLE`）— 跳过
2. 检查是否已初始化或正在初始化
3. 重置失败状态（如果之前失败）
4. 创建管理器实例
5. 异步初始化（不阻塞启动）
6. 成功后注册被动通知处理器

**幂等性：**
- 已初始化/进行中 → 跳过
- 之前失败 → 重试

#### `reinitializeLspServerManager(): void`
强制重新初始化。

**使用场景：**
- 插件刷新后（issue #15521）
- 加载新插件的 LSP 服务器

**流程：**
1. 关闭旧实例（fire-and-forget）
2. 清除状态
3. 调用 `initializeLspServerManager()`

#### `getLspServerManager(): LSPServerManager | undefined`
获取管理器实例。

**行为：**
- 未初始化/失败/进行中 → `undefined`
- 成功 → 实例

#### `getInitializationStatus()`
获取初始化状态。

**返回：**
- `'not-started'` — 未开始
- `'pending'` — 进行中
- `'success'` — 成功
- `'failed'` — 失败（含错误）

#### `isLspConnected(): boolean`
检查是否有健康连接的服务器。

#### `waitForInitialization(): Promise<void>`
等待初始化完成。

#### `shutdownLspServerManager(): Promise<void>`
关闭管理器。

**流程：**
1. 调用 `manager.shutdown()`
2. 清除所有状态
3. 增加 generation（使进行中的初始化无效）
4. 错误记录但不传播

### 生成计数器

`initializationGeneration` 用于：
- 使旧的初始化 Promise 无效
- 防止竞态条件
- 支持重新初始化

## 设计要点

1. **单例模式** — 全局管理器实例
2. **异步初始化** — 不阻塞应用启动
3. **状态机** — 明确的状态转换
4. **生成计数** — 竞态条件防护
5. **重新初始化支持** — 插件刷新后重建

## 与其他文件的关系

- **被调用**: `main.tsx` 启动流程
- **依赖**: `LSPServerManager.ts`
- **关联**: `passiveFeedback.ts` 处理器注册

## 注意事项

- bare 模式完全跳过 LSP
- 初始化失败不阻止应用运行
- 关闭时错误仅记录，确保状态清理
- 重新初始化时旧实例尽力关闭
