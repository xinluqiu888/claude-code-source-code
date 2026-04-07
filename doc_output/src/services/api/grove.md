# grove.ts — Grove 隐私通知服务

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/api/grove.ts`
- **所属模块**: API Service
- **功能类型**: 隐私设置和通知管理

## 功能概述

该模块管理 Grove 隐私通知功能，包括用户设置获取、通知显示逻辑和缓存管理。Grove 是通知用户隐私政策更新的功能。

## 核心内容详解

### 类型定义

#### `AccountSettings`
```typescript
{
  grove_enabled: boolean | null  // null = 用户未做选择
  grove_notice_viewed_at: string | null  // ISO 8601 时间戳
}
```

#### `GroveConfig`
```typescript
{
  grove_enabled: boolean           // 功能是否启用
  domain_excluded: boolean        // 域名是否被排除
  notice_is_grace_period: boolean // 是否在宽限期内
  notice_reminder_frequency: number | null  // 提醒频率（天）
}
```

#### `ApiResult<T>`
区分 API 失败和成功的结果类型：
- `{ success: true, data: T }`
- `{ success: false }`

### 主要函数

#### `getGroveSettings(): Promise<ApiResult<AccountSettings>>`
获取当前用户的 Grove 设置。

**端点：**
```
GET /api/oauth/account/settings
```

**特性：**
- 使用 memoize 会话级缓存
- 缓存失效：`updateGroveSettings()` 和 `markGroveNoticeViewed()` 调用后清除
- API 失败时清除缓存（避免网络问题导致设置不可用）

#### `markGroveNoticeViewed(): Promise<void>`
标记用户已查看 Grove 通知。

**端点：**
```
POST /api/oauth/account/grove_notice_viewed
```

#### `updateGroveSettings(groveEnabled): Promise<void>`
更新 Grove 设置。

**端点：**
```
PATCH /api/oauth/account/settings
Body: { grove_enabled: boolean }
```

#### `isQualifiedForGrove(): Promise<boolean>`
非阻塞检查用户是否有资格使用 Grove。

**缓存策略：**
- 使用 `groveConfigCache` 全局配置缓存
- 缓存有效期：24 小时
- 无缓存时后台获取，返回 false（当前会话不显示）
- 缓存过期时返回旧值并后台刷新

#### `getGroveNoticeConfig(): Promise<ApiResult<GroveConfig>>`
获取 Grove 通知配置。

**端点：**
```
GET /api/claude_code_grove
```

**特性：**
- 3 秒短超时（缓慢时跳过）
- 返回默认值处理缺失字段

#### `calculateShouldShowGrove(settingsResult, configResult, showIfAlreadyViewed): boolean`
计算是否应该显示 Grove 对话框。

**显示条件：**
1. API 调用成功（任一失败则隐藏）
2. 用户尚未做出选择（`grove_enabled === null`）
3. 或满足提醒条件（宽限期结束或达到提醒频率）

#### `checkGroveForNonInteractive(): Promise<void>`
非交互式会话的 Grove 检查。

**行为：**
- 宽限期内：显示信息消息，继续执行
- 宽限期后：显示错误消息，退出（状态码 1）

### 缓存机制

```
账户级缓存（每个 accountId）：
{
  grove_enabled: boolean
  timestamp: number
}
```

**策略：**
- 24 小时过期
- 冷启动返回 false（后台获取）
- 过期数据返回旧值并后台刷新

## 设计要点

1. **非阻塞设计** — 网络缓慢时不阻塞用户体验
2. **多层缓存** — 账户级缓存 + memoize 函数级缓存
3. **优雅降级** — API 失败时隐藏通知而非崩溃
4. **宽限期支持** — 区分信息性和强制性通知
5. **提醒频率** — 支持定期提醒机制

## 与其他文件的关系

- **被调用**: `main.tsx` 启动流程、隐私设置 UI
- **依赖**: `../../utils/auth.ts` 获取账户信息
- **关联**: `../../utils/gracefulShutdown.ts` 退出处理

## 注意事项

- 仅消费者订阅者（`isConsumerSubscriber()`）有资格
- 需要有效的 OAuth 访问令牌
- 非交互式会话的处理更严格（宽限期后退出）
- 调试日志使用 `logForDebugging` 前缀 `[Grove]`
