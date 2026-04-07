# diagLogs.ts — 诊断日志系统

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/diagLogs.ts`
- **主要功能**: 记录诊断信息用于监控和遥测（不含 PII）
- **关键依赖**: `fsOperations.ts`, `slowOperations.ts`

## 功能概述

该模块提供诊断日志功能，用于：
1. 记录诊断事件到日志文件
2. 包装函数添加诊断计时
3. 监控问题（不包含 PII）

**重要**: 该函数**不得**记录任何 PII，包括文件路径、项目名称、仓库名、提示等。

## 核心内容详解

### 诊断日志结构

```typescript
type DiagnosticLogLevel = 'debug' | 'info' | 'warn' | 'error'

type DiagnosticLogEntry = {
  timestamp: string
  level: DiagnosticLogLevel
  event: string
  data: Record<string, unknown>
}
```

### 核心函数

#### logForDiagnosticsNoPII

```typescript
export function logForDiagnosticsNoPII(
  level: DiagnosticLogLevel,
  event: string,
  data?: Record<string, unknown>
): void
```

- 同步 I/O（可从同步上下文调用）
- 写入 `CLAUDE_CODE_DIAGNOSTICS_FILE` 指定的文件
- 自动创建目录
- 静默失败（如果日志不可用）

#### withDiagnosticsTiming

```typescript
export async function withDiagnosticsTiming<T>(
  event: string,
  fn: () => Promise<T>,
  getData?: (result: T) => Record<string, unknown>
): Promise<T>
```

包装异步函数，自动记录：
- `{event}_started` - 执行前
- `{event}_completed` - 执行后（包含 duration_ms）
- `{event}_failed` - 执行失败

## 设计要点

1. **同步写入**: 使用同步文件操作支持同步上下文
2. **JSON Lines 格式**: 每行一个 JSON 对象便于解析
3. **PII 安全**: 明确禁止记录敏感信息
4. **错误恢复**: 失败时静默处理
5. **自动计时**: 包装函数自动计算持续时间

## 与其他文件的关系

| 文件 | 关系 |
|------|------|
| `fsOperations.ts` | 使用 `getFsImplementation` |
| `slowOperations.ts` | 使用 `jsonStringify` |

## 注意事项

1. **PII 安全**: 调用者必须确保不传入敏感信息
2. **环境变量**: `CLAUDE_CODE_DIAGNOSTICS_FILE` 控制日志文件路径
3. **同步写入**: 可能阻塞事件循环，避免高频调用
4. **错误静默**: 日志失败时不抛出错误
