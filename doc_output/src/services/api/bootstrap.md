# bootstrap.ts — 启动数据获取服务

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/api/bootstrap.ts`
- **所属模块**: API Service
- **功能类型**: 启动时数据获取

## 功能概述

该模块在应用启动时从 API 获取引导数据（bootstrap data），包括客户端数据和额外模型选项。支持 OAuth 和 API 密钥认证，并缓存结果到磁盘。

## 核心内容详解

### API 响应模式

```typescript
{
  client_data?: Record<string, unknown> | null
  additional_model_options?: Array<{
    model: string      // 模型 ID
    name: string       // 显示名称
    description: string // 描述
  }> | null
}
```

### 主要函数

#### `fetchBootstrapData(): Promise<void>`
获取引导数据并持久化到磁盘缓存。

**流程：**
1. 调用 `fetchBootstrapAPI()` 获取数据
2. 提取 `client_data` 和 `additional_model_options`
3. 比较现有缓存与新数据
4. 仅在数据变化时写入磁盘（避免不必要的写入）

**跳过条件：**
- 仅允许必要流量（`isEssentialTrafficOnly()`）
- 使用第三方提供商
- 无可用的 OAuth 或 API 密钥

#### `fetchBootstrapAPI(): Promise<BootstrapResponse | null>`
内部 API 调用实现。

**认证优先级：**
1. OAuth（需要 `user:profile` 范围）
2. API 密钥

**端点：**
- 生产：`https://api.anthropic.com/api/claude_cli/bootstrap`
- 测试：由 `ANTHROPIC_BASE_URL` 决定

**特性：**
- 使用 `withOAuth401Retry` 处理 OAuth 401 重试
- 5 秒超时
- Zod 响应验证

### 缓存策略

**存储位置：**
- `clientDataCache` — 全局配置的 `client_data` 缓存
- `additionalModelOptionsCache` — 额外模型选项缓存

**缓存更新逻辑：**
- 使用 `isEqual` 比较新旧数据
- 数据未变化时跳过写入
- 减少启动时的磁盘 I/O

## 设计要点

1. **条件获取** — 仅在必要时（非仅必要流量、First Party、有认证）获取
2. **认证回退** — OAuth 优先，API 密钥回退
3. **响应验证** — 使用 Zod 验证 API 响应结构
4. **智能缓存** — 仅在数据变化时写入磁盘
5. **调试日志** — 详细的获取状态日志

## 与其他文件的关系

- **被调用**: `main.tsx` 启动流程
- **依赖**: `../../constants/oauth.ts` 获取配置
- **关联**: `../../utils/config.ts` 全局配置管理

## 注意事项

- 引导数据失败不会阻止应用启动（失败时返回 null）
- 服务密钥 OAuth 令牌缺少 `user:profile` 范围会回退到 API 密钥
- API 密钥用户在 401 时不会重试（无刷新机制）
