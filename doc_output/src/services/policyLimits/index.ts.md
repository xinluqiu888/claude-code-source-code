# index.ts — 策略限制服务

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/policyLimits/index.ts`
- **作用域**: 组织级别策略限制的获取和应用
- **主要导出**:
  - `loadPolicyLimits`: 加载策略限制
  - `refreshPolicyLimits`: 刷新策略限制
  - `clearPolicyLimitsCache`: 清除策略限制缓存
  - `isPolicyAllowed`: 检查策略是否允许
  - `isPolicyLimitsEligible`: 检查用户是否符合策略限制条件
  - `waitForPolicyLimitsToLoad`: 等待策略限制加载完成
  - `initializePolicyLimitsLoadingPromise`: 初始化加载 Promise

## 功能概述

从 API 获取组织级别的策略限制，用于禁用 CLI 功能。遵循与远程管理设置相同的模式（fail open、ETag 缓存、后台轮询、重试逻辑）。

### 适用性

- Console 用户（API key）：全部符合
- OAuth 用户（Claude.ai）：仅 Team 和 Enterprise/C4E 订阅者符合
- API 失败开放（非阻塞）- 如果获取失败，继续无限制运行
- API 对没有策略限制的用户返回空限制

## 核心内容详解

### 常量配置

```typescript
const CACHE_FILENAME = 'policy-limits.json'
const FETCH_TIMEOUT_MS = 10000          // 10 秒
const DEFAULT_MAX_RETRIES = 5
const POLLING_INTERVAL_MS = 60 * 60 * 1000  // 1 小时
const LOADING_PROMISE_TIMEOUT_MS = 30000    // 30 秒
```

### 核心函数

#### `isPolicyLimitsEligible()`
检查用户是否符合策略限制条件：
- 第三方 provider 用户：不符合
- 自定义 base URL 用户：不符合
- Console 用户（API key）：符合
- OAuth 用户（Claude.ai）：
  - 必须有 `CLAUDE_AI_INFERENCE_SCOPE`
  - 必须是 Team 或 Enterprise 订阅

#### `loadPolicyLimits()`
CLI 初始化期间加载策略限制：
1. 检查用户是否符合条件
2. 从缓存加载现有限制
3. 从 API 获取最新限制（带重试）
4. 处理 304 Not Modified（缓存有效）
5. 保存新限制到缓存
6. 启动后台轮询
7. 失败开放 - 获取失败时继续运行

#### `fetchWithRetry(cachedChecksum?)`
带重试逻辑的策略限制获取：
- 最多 5 次重试
- 指数退避延迟
- 支持 HTTP ETag 缓存验证

#### `fetchPolicyLimits(cachedChecksum?)`
单次策略限制获取：
- 支持 API key 和 OAuth 认证
- 发送 `If-None-Match` 头部（如有缓存校验和）
- 处理 200、304、404 状态码
- 使用 Zod Schema 验证响应

#### `isPolicyAllowed(policy)`
检查特定策略是否允许：
- 从缓存获取限制
- 策略不存在 = 允许
- 明确 `allowed: false` = 阻止
- 失败开放（默认允许）
- **例外**: `allow_product_feedback` 在 essential-traffic-only 模式下失败关闭

### 缓存机制

#### `computeChecksum(restrictions)`
计算限制内容的 SHA256 校验和用于 HTTP 缓存

#### `loadCachedRestrictions()`
从文件同步加载缓存的限制（sessionCache -> 文件缓存）

#### `saveCachedRestrictions(restrictions)`
异步保存限制到缓存文件（权限 0o600）

### 后台轮询

#### `startBackgroundPolling()`
启动每小时轮询以检测策略变化：
- 间隔：1 小时
- 使用 `unref()` 避免阻塞进程退出
- 注册 cleanup handler

#### `pollPolicyLimits()`
轮询回调：
- 检查用户是否符合条件
- 获取最新限制
- 检测变化并记录日志
- 失败不阻塞

### 辅助函数

#### `getAuthHeaders()`
获取认证头部（支持 API key 和 OAuth）

#### `sortKeysDeep(obj)`
递归排序对象键以确保一致的哈希

#### `_resetPolicyLimitsForTesting()`
测试专用同步重置

## 设计要点

1. **失败开放**: 获取失败时继续无限制运行
2. **ETag 缓存**: 使用 HTTP ETag 和校验和进行高效缓存
3. **后台轮询**: 每小时轮询检测策略变化
4. **双重认证**: 支持 API key 和 OAuth
5. ** eligibility 检查**: 严格的用户资格验证
6. **HIPAA 保护**: `allow_product_feedback` 在 essential-traffic-only 模式下失败关闭

## 与其他文件的关系

- **types.ts**: 类型定义和 Zod Schema
- **auth.ts**: 认证相关函数
- **oauth.ts**: OAuth 配置和常量
- **withRetry.ts**: 重试延迟计算

## 注意事项

1. **避免循环依赖**: `isPolicyLimitsEligible` 不能调用 `getSettings()`
2. **同步缓存加载**: `loadCachedRestrictions` 是同步的
3. **加载 Promise 超时**: 30 秒超时防止死锁
4. **隐私级别**: `isEssentialTrafficOnly()` 影响 `allow_product_feedback` 行为
5. **文件权限**: 缓存文件使用 0o600 权限
