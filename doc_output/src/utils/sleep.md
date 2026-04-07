# sleep.ts — 异步延时和超时工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/sleep.ts`
- **主要功能**: 支持中止信号的睡眠函数和 Promise 超时竞速
- **关键依赖**: 无

## 功能概述

该模块提供两个核心异步工具函数：
1. `sleep` - 支持 AbortSignal 的延时函数
2. `withTimeout` - 为 Promise 添加超时限制

## 核心内容详解

### sleep 函数

```typescript
export function sleep(
  ms: number,
  signal?: AbortSignal,
  opts?: { 
    throwOnAbort?: boolean; 
    abortError?: () => Error; 
    unref?: boolean 
  }
): Promise<void>
```

#### 功能特性

1. **中止响应**: 当 `signal` 中止时立即解析
2. **中止处理模式**:
   - 默认: 静默解析（调用者检查 `signal.aborted`）
   - `throwOnAbort: true`: 中止时拒绝
   - `abortError`: 自定义拒绝错误
3. **unref 选项**: 不阻止进程退出（用于后台任务）

#### 实现细节

```typescript
// 关键实现细节
if (signal?.aborted) {
  if (opts?.throwOnAbort || opts?.abortError) {
    void reject(opts.abortError?.() ?? new Error('aborted'))
  } else {
    void resolve()
  }
  return
}
```

- 在设置定时器前检查中止状态（避免临时死区问题）
- 使用 `once: true` 确保只触发一次
- 清理事件监听器避免内存泄漏

### withTimeout 函数

```typescript
export function withTimeout<T>(
  promise: Promise<T>,
  ms: number,
  message: string
): Promise<T>
```

#### 功能特性

1. **竞速模式**: Promise 与定时器竞速，先完成者获胜
2. **自动清理**: Promise 完成后清除定时器
3. **unref**: 定时器不阻止进程退出
4. **注意**: 不会取消底层操作

#### 实现细节

```typescript
return Promise.race([promise, timeoutPromise]).finally(() => {
  if (timer !== undefined) clearTimeout(timer)
})
```

## 设计要点

1. **AbortSignal 兼容**: 符合现代 Web 标准的取消模式
2. **零依赖**: 纯原生实现
3. **内存安全**: 正确清理事件监听器和定时器
4. **灵活性**: 多种中止处理模式满足不同场景
5. **竞速安全**: `withTimeout` 确保无悬挂定时器

## 与其他文件的关系

该模块无外部依赖，可被任何模块安全导入。

## 注意事项

1. **signal 检查**: 中止后静默解析，调用者需检查 `signal.aborted`
2. **withTimeout 限制**: 仅返回控制权，不取消底层异步操作
3. **错误消息**: `withTimeout` 的超时消息用于 `Error(message)`
4. **unref 使用**: 仅在需要允许进程退出时使用（如后台任务）
5. **V8 优化**: 避免在 `onAbort` 回调中引用尚未定义的变量
