# referral.ts — 推荐计划服务

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/api/referral.ts`
- **所属模块**: API Service
- **功能类型**: 推荐计划管理

## 功能概述

该模块管理 Claude Code 的推荐计划功能，包括获取推荐资格、缓存推荐数据和格式化奖励显示。主要支持 "Guest Pass" 推荐活动。

## 核心内容详解

### 常量配置

```typescript
CACHE_EXPIRATION_MS = 24 * 60 * 60 * 1000  // 24 小时缓存
```

### 类型定义

#### `ReferralCampaign`
推荐活动类型：
- `'claude_code_guest_pass'` — Claude Code 访客通行证

#### `ReferralEligibilityResponse`
资格响应：
```typescript
{
  eligible: boolean           // 是否有资格
  remaining_passes: number    // 剩余通行证数
  campaign_version: string    // 活动版本
  referrer_reward?: ReferrerRewardInfo  // 推荐人奖励
}
```

#### `ReferrerRewardInfo`
奖励信息：
```typescript
{
  amount_minor_units: number  // 金额（最小单位）
  currency: string            // 货币代码
}
```

### 主要函数

#### `fetchReferralEligibility(campaign): Promise<ReferralEligibilityResponse>`
获取推荐资格。

**端点：**
```
GET /api/oauth/organizations/{orgUUID}/referral/eligibility?campaign={campaign}
```

**参数：**
- `campaign` — 活动名称（默认：claude_code_guest_pass）

#### `fetchReferralRedemptions(campaign): Promise<ReferralRedemptionsResponse>`
获取推荐兑换记录。

**端点：**
```
GET /api/oauth/organizations/{orgUUID}/referral/redemptions
```

### 缓存管理

#### `checkCachedPassesEligibility(): { eligible, needsRefresh, hasCache }`
检查缓存的资格状态。

**返回：**
- `eligible` — 缓存的资格状态
- `needsRefresh` — 是否需要刷新
- `hasCache` — 是否有缓存

#### `getCachedOrFetchPassesEligibility(): Promise<ReferralEligibilityResponse | null>`
获取缓存或获取新的资格数据。

**非阻塞策略：**
- 无缓存 — 触发后台获取，返回 null（当前会话不可用）
- 缓存过期 — 返回旧值，后台刷新
- 缓存新鲜 — 立即返回

#### `fetchAndStorePassesEligibility(): Promise<ReferralEligibilityResponse | null>`
获取并存储资格数据。

**去重机制：**
- 使用 `fetchInProgress` 防止重复请求
- 并发调用共享同一 Promise

### 辅助函数

#### `shouldCheckForPasses(): boolean`
检查是否应该查询通行证资格。

**条件：**
- 有组织 UUID
- 是 Claude AI 订阅者
- 订阅类型为 Max

#### `getCachedReferrerReward(): ReferrerRewardInfo | null`
从缓存获取推荐人奖励信息。

#### `getCachedRemainingPasses(): number | null`
从缓存获取剩余通行证数。

#### `formatCreditAmount(reward): string`
格式化奖励金额为可读字符串。

**货币符号映射：**
- USD: $
- EUR: €
- GBP: £
- BRL: R$
- CAD: CA$
- AUD: A$
- NZD: NZ$
- SGD: S$

### 预取

#### `prefetchPassesEligibility(): Promise<void>`
启动时预取资格数据。

**条件：**
- 跳过仅必要流量
- 异步执行，不阻塞

## 设计要点

1. **非阻塞设计** — 网络缓慢时不阻塞用户体验
2. **请求去重** — 并发调用共享同一请求
3. **智能缓存** — 24 小时有效期，过期数据可用
4. **货币本地化** — 支持多种货币的符号显示
5. **资格门控** — 仅 Max 订阅者可用

## 与其他文件的关系

- **被调用**: 启动流程、推荐命令 UI
- **依赖**: `../../utils/teleport/api.ts` 的 OAuth 工具
- **关联**: `../oauth/types.ts` 类型定义

## 注意事项

- 仅组织账户的 Max 订阅者有推荐资格
- 缓存按组织 UUID 存储
- 网络错误时返回 null，不中断流程
- 调试日志使用 `logForDebugging` 前缀 `Passes:`
