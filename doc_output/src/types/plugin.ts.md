# plugin.ts — 插件系统类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/types/plugin.ts`
- **类型**: TypeScript 模块
- **导出内容**: 插件定义、插件配置、插件加载结果、插件错误类型等
- **依赖关系**:
  - 导入: `lsp/types.js`, `mcp/types.js`, `bundledSkills.js`, `plugins/schemas.js`, `settings/types.js`

## 功能概述

本文件定义了 Claude Code 插件系统的完整类型体系，包括内置插件定义、插件仓库配置、加载后的插件结构，以及丰富的插件错误类型。它支持插件的版本管理、多来源加载（内置、市场、MCP、LSP 等）和详细的错误报告。

## 核心内容详解

### 1. BuiltinPluginDefinition (第18-35行)

内置插件定义，随 CLI 一起发布的插件：

```typescript
export type BuiltinPluginDefinition = {
  name: string                   // 插件名称（用于 {name}@builtin 标识）
  description: string            // 描述（显示在 /plugin UI）
  version?: string               // 可选版本
  skills?: BundledSkillDefinition[]      // 提供的技能
  hooks?: HooksSettings          // 提供的钩子
  mcpServers?: Record<string, McpServerConfig>  // MCP 服务器
  isAvailable?: () => boolean    // 是否可用（基于系统能力）
  defaultEnabled?: boolean       // 默认启用状态
}
```

**特点**:
- 用户可以在 /plugin UI 中启用/禁用
- 设置持久化到用户配置
- 不可用的插件完全隐藏

### 2. PluginRepository (第37-42行)

插件仓库配置：

```typescript
export type PluginRepository = {
  url: string
  branch: string
  lastUpdated?: string
  commitSha?: string
}
```

### 3. PluginConfig (第44-47行)

插件配置根结构：

```typescript
export type PluginConfig = {
  repositories: Record<string, PluginRepository>
}
```

### 4. LoadedPlugin (第48-70行)

加载后的插件结构：

```typescript
export type LoadedPlugin = {
  name: string
  manifest: PluginManifest
  path: string
  source: string
  repository: string
  enabled?: boolean
  isBuiltin?: boolean            // 是否是内置插件
  sha?: string                   // Git commit SHA（版本固定）
  commandsPath?: string
  commandsPaths?: string[]       // 额外命令路径
  commandsMetadata?: Record<string, CommandMetadata>
  agentsPath?: string
  agentsPaths?: string[]
  skillsPath?: string
  skillsPaths?: string[]
  outputStylesPath?: string
  outputStylesPaths?: string[]
  hooksConfig?: HooksSettings
  mcpServers?: Record<string, McpServerConfig>
  lspServers?: Record<string, LspServerConfig>
  settings?: Record<string, unknown>
}
```

**支持的组件路径**:
- `commandsPaths`: 命令路径
- `agentsPaths`: 代理定义路径
- `skillsPaths`: 技能路径
- `outputStylesPaths`: 输出样式路径

### 5. PluginComponent (第72-78行)

插件组件类型：

```typescript
export type PluginComponent =
  | 'commands'
  | 'agents'
  | 'skills'
  | 'hooks'
  | 'output-styles'
```

### 6. PluginError 联合类型 (第80-283行)

丰富的插件错误类型系统（20+ 种错误类型）：

#### 已实现类型（生产中使用）:

| 错误类型 | 说明 |
|----------|------|
| `generic-error` | 通用插件加载失败 |
| `plugin-not-found` | 在市场中未找到插件 |

#### 计划中的类型（未来使用）:

| 错误类型 | 说明 |
|----------|------|
| `path-not-found` | 组件路径不存在 |
| `git-auth-failed` | Git 认证失败 |
| `git-timeout` | Git 操作超时 |
| `network-error` | 网络错误 |
| `manifest-parse-error` | 清单解析错误 |
| `manifest-validation-error` | 清单验证失败 |
| `marketplace-not-found` | 市场未找到 |
| `marketplace-load-failed` | 市场加载失败 |
| `mcp-config-invalid` | MCP 配置无效 |
| `mcp-server-suppressed-duplicate` | MCP 服务器重复被抑制 |
| `hook-load-failed` | 钩子加载失败 |
| `component-load-failed` | 组件加载失败 |
| `mcpb-download-failed` | MCPB 下载失败 |
| `mcpb-extract-failed` | MCPB 解压失败 |
| `mcpb-invalid-manifest` | MCPB 清单无效 |
| `lsp-config-invalid` | LSP 配置无效 |
| `lsp-server-start-failed` | LSP 服务器启动失败 |
| `lsp-server-crashed` | LSP 服务器崩溃 |
| `lsp-request-timeout` | LSP 请求超时 |
| `lsp-request-failed` | LSP 请求失败 |
| `marketplace-blocked-by-policy` | 市场被企业策略阻止 |
| `dependency-unsatisfied` | 依赖未满足 |
| `plugin-cache-miss` | 插件缓存未命中 |

### 7. PluginLoadResult (第285-289行)

插件加载结果：

```typescript
export type PluginLoadResult = {
  enabled: LoadedPlugin[]
  disabled: LoadedPlugin[]
  errors: PluginError[]
}
```

### 8. getPluginErrorMessage() (第295-362行)

从 PluginError 获取显示消息的辅助函数：

**实现特点**:
- 为每种错误类型提供特定的格式化消息
- 支持变量插值（如插件名、路径等）
- 为用户提供可操作的提示（如依赖未满足时的建议）

**示例**:
```typescript
case 'mcp-server-suppressed-duplicate': {
  const dup = error.duplicateOf.startsWith('plugin:')
    ? `server provided by plugin "${error.duplicateOf.split(':')[1] ?? '?'}":'
    : `already-configured "${error.duplicateOf}"`
  return `MCP server "${error.serverName}" skipped — same command/URL as ${dup}`
}
```

## 设计要点

1. **类型安全错误**: 使用 discriminated union 替代字符串匹配
2. **丰富上下文**: 每种错误类型都有特定的上下文字段
3. **用户友好**: getPluginErrorMessage 提供格式化的用户消息
4. **扩展性**: 已预留 20+ 错误类型，可增量实现
5. **多组件支持**: 插件可以包含命令、代理、技能、钩子、样式等

## 与其他文件的关系

- **utils/plugins/schemas.ts**: 提供 PluginManifest 等类型
- **utils/plugins/pluginLoader.ts**: 使用这些类型进行加载
- **commands/plugin/**: 使用这些类型显示插件信息
- **services/mcp/types.ts**: 使用 McpServerConfig
- **services/lsp/types.ts**: 使用 LspServerConfig

## 使用场景

```typescript
// 定义内置插件
const builtinPlugin: BuiltinPluginDefinition = {
  name: 'my-builtin',
  description: 'My built-in plugin',
  skills: [...],
  defaultEnabled: true,
}

// 处理插件加载结果
const result: PluginLoadResult = await loadPlugins()
for (const error of result.errors) {
  console.error(getPluginErrorMessage(error))
}

// 启用插件
for (const plugin of result.enabled) {
  registerCommands(plugin.commandsPath)
}
```

## 注意事项

1. **错误类型增量实现**: 目前有 2 种在生产中使用，其他为未来预留
2. **路径复数形式**: commandsPaths 等复数字段支持多个路径
3. **版本固定**: sha 字段支持 Git commit SHA 版本固定
4. **内置插件标记**: isBuiltin 区分随 CLI 发布的插件
5. **MCPB 支持**: 支持 MCPB（MCP Bundle）格式插件
