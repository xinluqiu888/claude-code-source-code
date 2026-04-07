# withRetry.ts — API 重试逻辑

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/api/withRetry.ts`
- **所属模块**: API Service
- **功能类型**: API 请求重试和错误处理

## 功能概述

该模块提供强大的 API 请求重试机制，支持多种错误类型的智能重试、指数退避、模型回退和持久化重试模式。

## 核心内容详解

### 常量配置

```typescript
DEFAULT_MAX_RETRIES = 10              // 默认最大重试次数
FLOOR_OUTPUT_TOKENS = 3000            // 最小输出 Token 数
MAX_529_RETRIES = 3                   // 529 错误最大重试次数
BASE_DELAY_MS = 500                   // 基础延迟（毫秒）
PERSISTENT_MAX_BACKOFF_MS = 5 * 60 * 1000    // 持久模式最大退避（5分钟）
PERSISTENT_RESET_CAP_MS = 6 * 60 * 60 * 1000 // 持久模式重置上限（6小时）
HEARTBEAT_INTERVAL_MS = 30_000        // 心跳间隔（30秒）
```

### 错误类

#### `CannotRetryError`
不可重试错误包装器。
- 保留原始错误和重试上下文
- 保留原始堆栈跟踪

#### `FallbackTriggeredError`
模型回退触发错误。
- 记录原始模型和回退模型

### 重试上下文

#### `RetryContext`
重试期间传递的上下文：
```typescript
{
  maxTokensOverride?: number  // Token 溢出时的覆盖值
  model: string               // 当前模型
  thinkingConfig: ThinkingConfig  // 思考配置
  fastMode?: boolean          // Fast 模式状态
}
```

### 前台查询源

#### `FOREGROUND_529_RETRY_SOURCES`
支持 529 重试的前台查询源集合：
- `repl_main_thread` 及其变体
- `sdk`
- `agent:*`
- `compact`
- `hook_agent`, `hook_prompt`
- `verification_agent`, `side_question`
- `auto_mode`
- `bash_classifier`（ant-only）

后台查询（摘要、标题等）在 529 时立即失败，避免级联放大。

### 主要函数

#### `withRetry<T>(getClient, operation, options): AsyncGenerator<SystemAPIErrorMessage, T>`
带重试的 API 操作执行器。

**参数：**
- `getClient` — 获取 Anthropic 客户端的函数
- `operation` — 实际 API 操作
- `options` — 重试选项

**重试选项：**
- `maxRetries` — 最大重试次数
- `model` — 当前模型
- `fallbackModel` — 回退模型
- `thinkingConfig` — 思考配置
- `fastMode` — Fast 模式
- `signal` — 中止信号
- `querySource` — 查询来源
- `initialConsecutive529Errors` — 初始连续 529 错误数

**重试逻辑：**

1. **Fast 模式回退**
   - 429/529 错误时检查 retry-after
   - 短时间（<20秒）→ 等待并重试
   - 长时间 → 进入冷却（切换到标准速度）
   - 超额使用禁用 → 永久禁用 Fast 模式

2. **529 错误处理**
   - 后台查询立即失败
   - 前台查询计数连续 529
   - 达到 `MAX_529_RETRIES` → 触发模型回退
   - 外部用户显示自定义过载错误

3. **认证错误处理**
   - 401 → 刷新 OAuth 令牌
   - 403 "token revoked" → 刷新令牌
   - Bedrock/GCP 认证错误 → 清除凭证缓存
   - ECONNRESET/EPIPE → 禁用 keep-alive

4. **Token 溢出处理**
   - 解析 400 错误中的溢出信息
   - 计算可用上下文
   - 调整 `maxTokensOverride`
   - 继续重试

5. **持久化模式**
   - 环境变量 `CLAUDE_CODE_UNATTENDED_RETRY` 启用
   - 429/529 无限重试
   - 高退避（最大 5 分钟，封顶 6 小时）
   - 心跳保持（每 30 秒输出进度）
   - 窗口限制使用 reset 时间戳等待

### 重试决策

#### `shouldRetry(error): boolean`
决定是否应该重试错误。

**检查顺序：**
1. Mock 错误 — 永不重试
2. 持久模式容量错误 — 总是重试
3. CCR 模式 401/403 — 总是重试
4. 过载错误（消息包含）— 重试
5. Token 溢出 — 重试
6. x-should-retry 头 — 遵循服务器指示
7. 连接错误 — 重试
8. 408/409 — 重试
9. 429 — 非订阅者/企业用户重试
10. 401 — 清除缓存并重试
11. 403 token revoked — 重试
12. 5xx — 重试

### 延迟计算

#### `getRetryDelay(attempt, retryAfterHeader?, maxDelayMs?): number`
计算重试延迟。

**逻辑：**
- 如果有 retry-after 头 → 使用其值
- 否则指数退避：`BASE_DELAY_MS * 2^(attempt-1)`
- 最大延迟封顶（默认 32 秒，持久模式 5 分钟）
- 添加 25% 抖动

#### `getRateLimitResetDelayMs(error): number | null`
从 rate limit reset 头计算等待时间。

### 错误解析

#### `parseMaxTokensContextOverflowError(error)`
解析 400 错误中的 Token 溢出信息。

**格式：**
```
input length and `max_tokens` exceed context limit: {inputTokens} + {maxTokens} > {contextLimit}
```

#### `is529Error(error): boolean`
检测 529 过载错误。

**检查：**
- 状态码 529
- 或错误消息包含 `"type":"overloaded_error"`

#### `isFastModeNotEnabledError(error): boolean`
检测 Fast 模式未启用错误。

#### `isOAuthTokenRevokedError(error): boolean`
检测 OAuth 令牌被撤销错误。

#### `isBedrockAuthError(error): boolean`
检测 AWS Bedrock 认证错误。

#### `isVertexAuthError(error): boolean`
检测 Google Vertex 认证错误。

## 设计要点

1. **生成器模式** — 允许在重试间输出系统消息
2. **上下文传递** — 在重试间保持和调整配置
3. **多种回退** — Fast 模式、模型、Token 溢出
4. **智能延迟** — 指数退避 + 抖动 + 服务器指示
5. **持久化支持** — 无人值守会话的无限重试
6. **认证恢复** — 自动刷新令牌和清除缓存

## 与其他文件的关系

- **被调用**: `claude.ts` 的流式和非流式 API 调用
- **依赖**: `../../utils/auth.ts` 认证管理
- **关联**: `../../utils/fastMode.ts` Fast 模式控制

## 注意事项

- 前台/后台区分避免级联放大
- 持久模式需要 `UNATTENDED_RETRY` 特性标志 + 环境变量
- 心跳输出通过生成器 yield 到 stdout
- 模型回退仅针对特定模型组合（非订阅者 Opus）
