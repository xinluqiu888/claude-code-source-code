# debug.ts — 调试日志系统

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/debug.ts`
- **主要功能**: 调试日志记录、分级日志、文件/stderr输出
- **关键依赖**: `fs/promises`, `lodash-es/memoize`, `bufferedWriter.ts`, `debugFilter.ts`, `envUtils.ts`

## 功能概述

该模块提供 Claude Code 的调试日志系统：
1. 多级日志记录（verbose/debug/info/warn/error）
2. 支持文件输出和 stderr 输出
3. 日志级别过滤和模式过滤
4. 异步缓冲写入器
5. 自动创建 `latest` 符号链接
6. Ant 专用错误日志

## 核心内容详解

### 日志级别

```typescript
export type DebugLogLevel = 'verbose' | 'debug' | 'info' | 'warn' | 'error'

const LEVEL_ORDER: Record<DebugLogLevel, number> = {
  verbose: 0,
  debug: 1,
  info: 2,
  warn: 3,
  error: 4,
}
```

### 调试模式检测

```typescript
export const isDebugMode = memoize((): boolean => {
  return (
    runtimeDebugEnabled ||
    isEnvTruthy(process.env.DEBUG) ||
    isEnvTruthy(process.env.DEBUG_SDK) ||
    process.argv.includes('--debug') ||
    process.argv.includes('-d') ||
    isDebugToStdErr() ||
    process.argv.some(arg => arg.startsWith('--debug=')) ||
    getDebugFilePath() !== null
  )
})
```

### 核心函数

#### `logForDebugging(message, { level })`
- 主要日志记录函数
- 支持多行消息自动转换为 JSON
- 根据配置输出到 stderr 或文件

#### `getMinDebugLogLevel(): DebugLogLevel`
- 从 `CLAUDE_CODE_DEBUG_LOG_LEVEL` 环境变量读取
- 默认值为 `'debug'`（过滤 verbose）

#### `enableDebugLogging(): boolean`
- 在会话中启用调试日志（如通过 `/debug` 命令）
- 返回之前是否已激活

#### `flushDebugLogs(): Promise<void>`
- 强制刷新缓冲的调试日志
- 用于确保日志在关键操作前写入

#### `logAntError(context, error): void`
- 仅对内部用户（`USER_TYPE=ant`）记录错误
- 包含完整堆栈跟踪

### 缓冲写入器

```typescript
let debugWriter: BufferedWriter | null = null
let pendingWrite: Promise<void> = Promise.resolve()
```

- `immediateMode`: 调试模式下同步写入
- `bufferedMode`: 非调试模式下缓冲写入（约1秒刷新）
- 使用 `BufferedWriter` 避免阻塞事件循环

### 日志文件路径

```typescript
export function getDebugLogPath(): string {
  return (
    getDebugFilePath() ??
    process.env.CLAUDE_CODE_DEBUG_LOGS_DIR ??
    join(getClaudeConfigHomeDir(), 'debug', `${getSessionId()}.txt`)
  )
}
```

## 设计要点

1. **分级日志**: 5个级别，支持级别过滤
2. **模式过滤**: `--debug=pattern` 支持按内容过滤
3. **双重输出**: 支持文件和 stderr 同时/单独输出
4. **缓冲策略**: 根据模式选择同步或异步写入
5. **符号链接**: 自动更新 `~/.claude/debug/latest` 指向当前日志
6. **Ant 专用**: 内部用户始终记录调试日志

## 与其他文件的关系

| 文件 | 关系 |
|------|------|
| `bufferedWriter.ts` | 提供 `BufferedWriter` 类 |
| `debugFilter.ts` | 日志过滤逻辑 |
| `envUtils.ts` | 使用 `getClaudeConfigHomeDir`, `isEnvTruthy` |
| `cleanupRegistry.ts` | 注册清理回调刷新日志 |
| `fsOperations.ts` | 获取文件系统实现 |
| `slowOperations.ts` | 使用 `jsonStringify` |

## 注意事项

1. **日志安全**: 多行消息自动 JSON 序列化以保持 JSONL 格式
2. **进程退出**: 同步模式确保日志在 `process.exit()` 前写入
3. **内存使用**: 缓冲模式可能累积未写入的日志
4. **环境变量**: `CLAUDE_CODE_DEBUG_LOG_LEVEL` 控制最小级别
5. **测试模式**: `NODE_ENV === 'test'` 时不写入日志（除非 `--debug-to-stderr`）
