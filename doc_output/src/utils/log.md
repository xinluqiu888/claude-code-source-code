# log.ts — 错误日志和 MCP 日志管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/log.ts`
- **主要功能**: 错误日志记录、MCP 日志、API 请求捕获、日志列表加载
- **关键依赖**: `fs/promises`, `bootstrap/state.js`, `types/logs.js`, `cachePaths.ts`

## 功能概述

该模块提供 Claude Code 的错误日志和诊断日志管理：
1. 错误日志记录（内存、文件、调试日志）
2. MCP 错误和调试日志
3. API 请求捕获（用于 bug 报告）
4. 错误日志列表加载
5. 日志显示标题生成

## 核心内容详解

### 错误日志接口

```typescript
export type ErrorLogSink = {
  logError: (error: Error) => void
  logMCPError: (serverName: string, error: unknown) => void
  logMCPDebug: (serverName: string, message: string) => void
  getErrorsPath: () => string
  getMCPLogsPath: (serverName: string) => string
}
```

### 核心函数

#### `logError(error): void`
- 记录错误到多个目的地
- 记录到调试日志、内存日志、持久化日志文件（Ant用户）
- 支持 `--hard-fail` 模式（进程退出）
- 跳过云提供商环境（Bedrock/Vertex/Foundry）

#### `attachErrorLogSink(sink): void`
- 附加错误日志接收器（Sink）
- 幂等操作（已附加时无操作）
- 立即排空队列中的待处理事件

#### `logMCPError(serverName, error): void`
- 记录 MCP 服务器错误
- 如果 Sink 未附加则排队

#### `logMCPDebug(serverName, message): void`
- 记录 MCP 调试消息

#### `captureAPIRequest(params, querySource): void`
- 捕获最后 API 请求用于 bug 报告
- 仅存储参数（不含消息）避免保留完整对话
- Ant 用户额外保存消息数组

#### `getInMemoryErrors(): Array`
- 获取内存中的最近错误
- 最多保留 `MAX_IN_MEMORY_ERRORS` (100) 条

### 日志列表加载

```typescript
export function loadErrorLogs(): Promise<LogOption[]>
export async function getErrorLogByIndex(index): Promise<LogOption | null>
```

- 从 `~/.claude/errors/` 加载错误日志
- 按日期排序
- 解析日志文件内容

### 显示标题生成

```typescript
export function getLogDisplayTitle(log, defaultTitle?): string
```

- 生成会话/日志的显示标题
- 跳过自动提示（tick/goal 标签）
- 剥离显示不友好的标签
- 回退到会话 ID 截断

## 设计要点

1. **多目标记录**: 错误同时记录到调试日志、内存和持久化存储
2. **Sink 模式**: 异步初始化时使用 Sink 模式，事件先排队后处理
3. **隐私保护**: API 请求默认不保存完整消息
4. **队列排空**: Sink 附加时立即处理排队事件
5. **Ant 专用**: 部分功能仅对内部用户启用

## 与其他文件的关系

| 文件 | 关系 |
|------|------|
| `bootstrap/state.js` | 使用状态管理函数 |
| `types/logs.js` | 导入日志类型 |
| `cachePaths.ts` | 使用 `CACHE_PATHS.errors()` |
| `displayTags.ts` | 使用 `stripDisplayTags` |
| `envUtils.ts` | 使用 `isEnvTruthy` |
| `errors.ts` | 使用 `toError` |
| `privacyLevel.ts` | 使用 `isEssentialTrafficOnly` |
| `slowOperations.ts` | 使用 `jsonParse` |

## 注意事项

1. **Sink 初始化**: 必须在应用启动时附加 Sink
2. **硬失败模式**: `--hard-fail` 会导致 `logError` 调用时进程退出
3. **内存限制**: 内存错误日志最多100条，旧条目被移除
4. **云环境**: 在云提供商环境中禁用错误报告
5. **API 请求**: 消息数组很大，默认不保留以节省内存
