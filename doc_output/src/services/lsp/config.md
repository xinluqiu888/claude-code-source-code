# config.ts — LSP 配置管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/lsp/config.ts`
- **所属模块**: LSP Service
- **功能类型**: 配置加载

## 功能概述

该模块从插件加载 LSP 服务器配置。LSP 服务器仅通过插件支持，不支持用户/项目级设置。

## 核心内容详解

### 主要函数

#### `getAllLspServers(): Promise<{ servers: Record<string, ScopedLspServerConfig> }>`
获取所有配置的 LSP 服务器。

**流程：**
1. 加载所有已启用插件（`loadAllPluginsCacheOnly`）
2. 并行从每个插件获取 LSP 服务器配置（`getPluginLspServers`）
3. 合并结果（后加载的插件优先）
4. 记录加载结果和错误

**错误处理：**
- 单个插件失败不影响其他插件
- 记录调试日志用于监控

### 依赖

- `loadAllPluginsCacheOnly` — 加载已启用插件
- `getPluginLspServers` — 从插件获取 LSP 配置

## 设计要点

1. **插件独占** — LSP 仅通过插件配置
2. **并行加载** — 提高启动性能
3. **错误隔离** — 单个插件失败不影响整体
4. **合并策略** — 后加载插件覆盖先加载的（Object.assign）

## 与其他文件的关系

- **被调用**: `LSPServerManager.ts` 初始化
- **依赖**: `pluginLoader.ts`, `lspPluginIntegration.ts`

## 注意事项

- 使用缓存的插件列表（`CacheOnly`）
- 配置范围由 `getPluginLspServers` 处理
