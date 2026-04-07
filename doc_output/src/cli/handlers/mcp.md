# handlers/mcp.tsx — MCP 子命令处理器

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/handlers/mcp.tsx` |
| **文件类型** | TypeScript/TSX 处理器模块 |
| **行数** | 约 500+ 行 |
| **职责** | 实现 `claude mcp` 子命令集，管理 MCP（Model Context Protocol）服务器 |

## 功能概述

本模块实现完整的 MCP 服务器管理功能，包括：

- **`serve`**：启动 MCP 服务器
- **`add`**：添加 MCP 服务器配置
- **`add-json`**：通过 JSON 字符串添加 MCP 服务器
- **`add-from-claude-desktop`**：从 Claude Desktop 导入 MCP 服务器
- **`remove`**：移除 MCP 服务器配置
- **`list`**：列出所有配置的 MCP 服务器
- **`get`**：查看特定 MCP 服务器的详细信息

MCP 是 Model Context Protocol 的缩写，是一种标准化协议，允许 Claude Code 与外部工具和服务集成。本模块提供完整的生命周期管理功能。

## 核心内容详解

### 导入

| 导入项 | 来源 | 用途 |
|--------|------|------|
| `stat` | `fs/promises` | 文件系统操作 |
| `pMap` | `p-map` | 并发映射处理 |
| `cwd` | `process` | 获取当前工作目录 |
| `React` | `react` | React 组件 |
| `MCPServerDesktopImportDialog` | `../../components/MCPServerDesktopImportDialog.js` | Desktop 导入对话框 |
| `render` | `../../ink.js` | Ink 渲染 |
| `KeybindingSetup` | `../../keybindings/KeybindingProviderSetup.js` | 快捷键设置 |
| `logEvent` | `../../services/analytics` | 分析事件记录 |
| `clearMcpClientConfig`, `clearServerTokensFromLocalStorage`, `getMcpClientConfig`, `readClientSecret`, `saveMcpClientSecret` | `../../services/mcp/auth.js` | MCP 认证相关 |
| `connectToServer`, `getMcpServerConnectionBatchSize` | `../../services/mcp/client.js` | MCP 客户端连接 |
| `addMcpConfig`, `getAllMcpConfigs`, `getMcpConfigByName`, `getMcpConfigsByScope`, `removeMcpConfig` | `../../services/mcp/config.js` | MCP 配置管理 |
| `ScopedMcpServerConfig`, `ConfigScope` | `../../services/mcp/types.js` | MCP 类型定义 |
| `describeMcpConfigFilePath`, `ensureConfigScope`, `getScopeLabel` | `../../services/mcp/utils.js` | MCP 工具函数 |
| `AppStateProvider` | `../../state/AppState.js` | 应用状态提供者 |
| `getCurrentProjectConfig`, `getGlobalConfig`, `saveCurrentProjectConfig` | `../../utils/config.js` | 配置管理 |
| `isFsInaccessible` | `../../utils/errors.js` | 文件系统错误检测 |
| `gracefulShutdown` | `../../utils/gracefulShutdown.js` | 优雅关闭 |
| `safeParseJSON` | `../../utils/json.js` | 安全 JSON 解析 |
| `getPlatform` | `../../utils/platform.js` | 平台检测 |
| `cliError`, `cliOk` | `../exit.js` | CLI 退出辅助 |

### 辅助函数

#### `async function checkMcpServerHealth(name, server): Promise<string>`

检查 MCP 服务器健康状态。

**返回值**：
- `'✓ Connected'` - 连接成功
- `'! Needs authentication'` - 需要认证
- `'✗ Failed to connect'` - 连接失败
- `'✗ Connection error'` - 连接错误

**实现**：
```typescript
const result = await connectToServer(name, server)
if (result.type === 'connected') return '✓ Connected'
else if (result.type === 'needs-auth') return '! Needs authentication'
else return '✗ Failed to connect'
```

### 核心处理器函数

#### `export async function mcpServeHandler({ debug, verbose })`

处理 `claude mcp serve` 命令。

**功能**：
- 启动 MCP 服务器模式
- 检查当前工作目录是否存在
- 调用 `setup()` 进行初始化
- 调用 `startMCPServer()` 启动服务器

**参数**：
- `debug`: 启用调试模式
- `verbose`: 启用详细输出

**错误处理**：
- 目录不存在：显示错误并退出
- 启动失败：显示错误信息

#### `export async function mcpRemoveHandler(name, options)`

处理 `claude mcp remove` 命令。

**功能**：
移除指定名称的 MCP 服务器配置，支持多作用域处理。

**逻辑流程**：

1. **查找配置**
   - 获取移除前的配置以便清理安全存储
   - 如果指定了 scope，直接在该 scope 中移除

2. **多作用域处理**
   - 检查服务器在哪些作用域中存在
   - 支持的作用域：`local`、`project`、`user`
   - 如果只存在于一个作用域，直接移除
   - 如果存在于多个作用域，提示用户指定 scope

3. **安全存储清理**
   ```typescript
   const cleanupSecureStorage = () => {
     if (serverBeforeRemoval?.type === 'sse' || serverBeforeRemoval?.type === 'http') {
       clearServerTokensFromLocalStorage(name, serverBeforeRemoval)
       clearMcpClientConfig(name, serverBeforeRemoval)
     }
   }
   ```

4. **成功响应**
   - 显示移除成功消息
   - 显示修改的文件路径
   - 使用 `cliOk()` 退出

**错误处理**：
- 服务器不存在：错误退出
- 多作用域冲突：提示用户使用 `-s` 指定 scope

#### `export async function mcpListHandler()`

处理 `claude mcp list` 命令。

**功能**：
列出所有配置的 MCP 服务器及其健康状态。

**执行流程**：

1. **获取配置**
   ```typescript
   const { servers: configs } = await getAllMcpConfigs()
   ```

2. **并发健康检查**
   ```typescript
   const results = await pMap(entries, async ([name, server]) => ({
     name,
     server,
     status: await checkMcpServerHealth(name, server)
   }), { concurrency: getMcpServerConnectionBatchSize() })
   ```

3. **格式化输出**
   - SSE 服务器：`{name}: {url} (SSE) - {status}`
   - HTTP 服务器：`{name}: {url} (HTTP) - {status}`
   - Claude.ai 代理：`{name}: {url} - {status}`
   - stdio 服务器：`{name}: {command} {args} - {status}`

**注意**：排除 `sse-ide` 类型的内部服务器

**退出**：使用 `gracefulShutdown(0)` 确保 MCP 连接正确清理

#### `export async function mcpGetHandler(name)`

处理 `claude mcp get` 命令。

**功能**：
显示特定 MCP 服务器的详细信息。

**显示内容**：
- 服务器名称
- 作用域（Scope）
- 健康状态
- 类型（sse/http/stdio）
- URL（sse/http）或命令参数（stdio）
- Headers（如有）
- OAuth 配置（如有）
- 环境变量（stdio 类型，如有）

**结尾提示**：显示移除命令示例

#### `export async function mcpAddHandler(name, command, args, options)`

处理 `claude mcp add` 命令。

**功能**：
添加新的 MCP 服务器配置。

**参数**：
- `name`: 服务器名称
- `command`: 命令（stdio 类型）或 URL（sse/http 类型）
- `args`: 命令参数数组
- `options`: 包含 scope、type、env、headers 等

**支持的类型**：
- `stdio`：标准输入输出（默认）
- `sse`：Server-Sent Events
- `http`：HTTP 流

**特殊处理**：
- 当类型为 `sse` 或 `http` 且有 `oauth.clientId` 时，交互式读取 `clientSecret`
- 验证 scope 有效性
- 保存配置到指定 scope

#### `export async function mcpAddJsonHandler(name, json, options)`

处理 `claude mcp add-json` 命令。

**功能**：
通过 JSON 字符串添加 MCP 服务器配置。

**参数**：
- `name`: 服务器名称
- `json`: JSON 配置字符串
- `options.scope`: 配置作用域
- `options.clientSecret`: 是否需要读取 client secret

**执行流程**：
1. 解析并验证 scope
2. 安全解析 JSON
3. 如果启用 `--client-secret`，交互式读取 secret
4. 添加配置
5. 如有 secret，保存到安全存储
6. 记录分析事件
7. 使用 `cliOk()` 显示成功

#### `export async function mcpAddFromDesktopHandler(options)`

处理 `claude mcp add-from-claude-desktop` 命令。

**功能**：
从 Claude Desktop 应用导入 MCP 服务器配置。

**执行流程**：
1. 验证 scope
2. 获取平台信息（macOS/Windows/Linux）
3. 读取 Claude Desktop 配置文件
4. 如果没有服务器，显示提示并退出
5. 渲染 `MCPServerDesktopImportDialog` React 组件
6. 允许用户选择要导入的服务器

**UI 流程**：
- 使用 Ink 渲染交互式对话框
- 包裹在 `AppStateProvider` 和 `KeybindingSetup` 中
- 支持 `exitOnCtrlC`

## 设计要点

1. **多作用域支持**：支持 `local`（项目本地）、`project`（.mcp.json）、`user`（全局）三种配置作用域
2. **并发健康检查**：使用 `pMap` 并发检查多个服务器状态，提高效率
3. **安全存储**：OAuth client secret 存储在安全存储中，不直接保存在配置文件
4. **优雅关闭**：使用 `gracefulShutdown` 而非 `process.exit`，确保 MCP 连接正确清理
5. **React 集成**：Desktop 导入使用 React 组件实现交互式 UI
6. **分析集成**：记录关键操作（add、delete、list 等）用于产品分析

## 与其他文件的关系

| 关系类型 | 文件 | 描述 |
|---------|------|------|
| **导入** | `src/services/mcp/config.js` | MCP 配置管理 |
| **导入** | `src/services/mcp/client.js` | MCP 客户端连接 |
| **导入** | `src/services/mcp/auth.js` | MCP 认证处理 |
| **导入** | `src/services/mcp/utils.js` | MCP 工具函数 |
| **导入** | `src/services/analytics` | 分析事件记录 |
| **导入** | `src/components/MCPServerDesktopImportDialog.js` | Desktop 导入对话框 |
| **导入** | `src/cli/exit.js` | CLI 退出辅助 |
| **被导入** | `src/main.tsx` | `claude mcp` 命令处理（动态导入）|
| **被导入** | `src/utils/claudeDesktop.js` | 读取 Desktop 配置（动态导入）|

## 注意事项

1. **动态导入**：模块设计为动态导入，仅在执行 `claude mcp` 命令时加载
2. **scope 验证**：使用 `ensureConfigScope()` 验证用户提供的 scope 参数
3. **错误处理**：使用 `cliError()` 和 `cliOk()` 统一错误处理和退出
4. **清理顺序**：移除服务器时先清理安全存储，再移除配置
5. **内部服务器排除**：`sse-ide` 类型的内部服务器在 list 和 get 中排除
6. **并发控制**：健康检查使用 `getMcpServerConnectionBatchSize()` 限制并发数
7. **类型安全**：使用 TypeScript 严格类型，特别是 MCP 配置类型
8. **交互式 secret 读取**：`readClientSecret()` 是交互式操作，在写入配置前进行
