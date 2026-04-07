# extra-usage-core.ts — 额外使用量核心逻辑

> **一句话总结**：处理额外使用量配置的核心业务逻辑，支持团队和企业的不同处理方式。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/extra-usage/extra-usage-core.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 118 行 |
| 主要职责 | 实现额外使用量配置的完整业务逻辑 |

---

## 功能概述

该文件实现了 `/extra-usage` 命令的核心业务逻辑，处理用户在达到使用限制时如何配置额外使用量。根据用户的订阅类型（个人/团队/企业）和权限，提供不同的处理路径：

1. **团队/企业用户（无账单权限）**: 需要向管理员发起请求
2. **团队/企业用户（有账单权限）**: 直接打开浏览器到管理页面
3. **个人用户**: 直接打开浏览器到设置页面

---

## 核心内容详解

### 导入与依赖

```typescript
import {
  checkAdminRequestEligibility,
  createAdminRequest,
  getMyAdminRequests,
} from '../../services/api/adminRequests.js'
import { invalidateOverageCreditGrantCache } from '../../services/api/overageCreditGrant.js'
import { type ExtraUsage, fetchUtilization } from '../../services/api/usage.js'
import { getSubscriptionType } from '../../utils/auth.js'
import { hasClaudeAiBillingAccess } from '../../utils/billing.js'
import { openBrowser } from '../../utils/browser.js'
import { getGlobalConfig, saveGlobalConfig } from '../../utils/config.js'
import { logError } from '../../utils/log.js'
```

### 类型定义

#### ExtraUsageResult

- **类型**: `union type`
- **变体**:
  - `{ type: 'message'; value: string }` - 返回文本消息
  - `{ type: 'browser-opened'; url: string; opened: boolean }` - 浏览器操作结果

### 主要函数

#### runExtraUsage(): Promise<ExtraUsageResult>

- **类型**: `async function`
- **返回值**: `Promise<ExtraUsageResult>`
- **用途**: 执行额外使用量配置的完整流程

**执行流程**:

1. **记录访问**: 如果是首次访问，更新全局配置 `hasVisitedExtraUsage`

2. **清除缓存**: 调用 `invalidateOverageCreditGrantCache()` 确保获取最新数据

3. **获取订阅信息**: 通过 `getSubscriptionType()` 获取用户订阅类型

4. **检查权限**: 通过 `hasClaudeAiBillingAccess()` 检查是否有账单管理权限

5. **无账单权限的团队/企业用户处理**:
   - 检查是否已有无限额外使用量（`extraUsage.is_enabled && monthly_limit === null`）
   - 检查管理员请求资格
   - 检查是否已有待处理或已驳回的请求
   - 创建新的管理员请求
   - 返回相应的消息

6. **有权限用户处理**:
   - 团队/企业用户打开 `https://claude.ai/admin-settings/usage`
   - 个人用户打开 `https://claude.ai/settings/usage`
   - 尝试用浏览器打开，捕获并记录错误

---

## 设计要点

1. **分层权限处理**: 根据用户订阅类型和账单权限提供不同的处理路径。

2. **防重复请求**: 检查是否已有待处理请求，避免用户重复提交。

3. **缓存管理**: 在操作前清除相关缓存，确保数据新鲜度。

4. **错误处理**: 所有API调用都有try-catch包裹，错误被记录但不中断流程。

5. **降级策略**: 即使浏览器打开失败，也会返回用户可手动访问的URL。

---

## 与其他文件的关系

**依赖**:
- Admin请求API (`../../services/api/adminRequests.js`)
- 超额授权缓存 (`../../services/api/overageCreditGrant.js`)
- 使用量API (`../../services/api/usage.js`)
- 认证工具 (`../../utils/auth.js`)
- 账单工具 (`../../utils/billing.js`)
- 浏览器工具 (`../../utils/browser.js`)
- 配置工具 (`../../utils/config.js`)

**被依赖**:
- `extra-usage.tsx` - 交互式版本
- `extra-usage-noninteractive.ts` - 非交互式版本

---

## 注意事项

1. **API错误处理**: 多个API调用使用try-catch包裹，即使失败也会优雅降级到通用消息。

2. **状态检查顺序**: 先检查是否已有无限使用量，再检查管理员资格，最后检查已有请求，逻辑顺序重要。

3. **URL区分**: 团队/企业用户和个人用户的管理页面URL不同。

4. **缓存失效**: 每次执行都清除超额授权缓存，确保获取最新状态。
