# errors.ts — 错误类型定义和工具函数

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/errors.ts`
- **主要功能**: 自定义错误类、错误类型判断、错误处理工具
- **关键依赖**: `@anthropic-ai/sdk`

## 功能概述

该模块定义 Claude Code 使用的各种错误类型和错误处理工具：
1. 自定义错误类（ClaudeError, AbortError, ShellError 等）
2. 错误类型判断函数（isAbortError, isENOENT, isFsInaccessible）
3. 错误信息提取工具（errorMessage, toError, shortErrorStack）
4. Axios 错误分类
5. 遥测安全错误（TelemetrySafeError）

## 核心内容详解

### 自定义错误类

#### `ClaudeError`
- 基础错误类，自动设置错误名称

#### `AbortError`
- 用户中止操作的错误
- `isAbortError()` 函数同时检查 SDK 的 `APIUserAbortError`

#### `ConfigParseError`
- 配置文件解析错误
- 包含文件路径和默认配置

#### `ShellError`
- Shell 命令执行失败
- 包含 stdout、stderr、退出码和中断状态

#### `TeleportOperationError`
- 目录切换操作错误
- 包含格式化消息用于显示

#### `TelemetrySafeError_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS`
- 标记为可安全发送到遥测的错误
- 确保消息不包含敏感信息（文件路径、代码片段）
- 支持用户消息和遥测消息分离

### 错误判断函数

#### `isAbortError(e): boolean`
```typescript
export function isAbortError(e: unknown): boolean {
  return (
    e instanceof AbortError ||
    e instanceof APIUserAbortError ||
    (e instanceof Error && e.name === 'AbortError')
  )
}
```

#### `isENOENT(e): boolean`
- 检查是否为文件不存在的错误

#### `isFsInaccessible(e)`
- 检查文件系统不可访问错误（ENOENT, EACCES, EPERM, ENOTDIR, ELOOP）
- 用于在 catch 块中区分预期错误

### 错误信息提取

#### `toError(e): Error`
- 将未知值转换为 Error 实例
- 用于 catch 边界

#### `errorMessage(e): string`
- 从错误中提取字符串消息

#### `shortErrorStack(e, maxFrames): string`
- 提取错误消息和前 N 个堆栈帧
- 用于工具结果中（节省上下文token）

#### `getErrnoCode(e): string | undefined`
- 提取错误代码（如 'ENOENT'）
- 替代 `(e as NodeJS.ErrnoException).code` 类型断言

#### `getErrnoPath(e): string | undefined`
- 提取文件系统路径

### Axios 错误分类

```typescript
export type AxiosErrorKind =
  | 'auth'     // 401/403
  | 'timeout'  // ECONNABORTED
  | 'network'  // ECONNREFUSED/ENOTFOUND
  | 'http'     // 其他HTTP错误
  | 'other'    // 非axios错误

export function classifyAxiosError(e): { kind, status?, message }
```

## 设计要点

1. **类型安全**: 使用 `unknown` 输入类型避免强制类型转换
2. **错误分类**: 细化的错误类型便于针对性处理
3. **遥测安全**: 显式标记可安全发送的错误
4. **性能优化**: `shortErrorStack` 限制堆栈帧数以节省token
5. **向后兼容**: `isAbortError` 检查多种中止错误来源

## 与其他文件的关系

| 文件 | 关系 |
|------|------|
| `@anthropic-ai/sdk` | 导入 `APIUserAbortError` |

## 注意事项

1. **isAbortError**: 使用 `instanceof` 检查 SDK 错误（minified 构建中类名被压缩）
2. **TelemetrySafeError**: 必须显式验证消息不包含敏感信息
3. **shortErrorStack**: 完整堆栈应保留在调试日志中
4. **Axios 检查**: 直接检查 `.isAxiosError` 属性避免依赖 axios 模块
