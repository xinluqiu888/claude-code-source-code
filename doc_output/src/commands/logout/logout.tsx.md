# logout.tsx — 用户登出流程

> **一句话总结**：执行完整的登出操作，清理所有认证相关缓存和凭证。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/logout/logout.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 81 行 |
| 主要职责 | 实现登出逻辑和认证缓存清理 |

---

## 功能概述

该文件实现了 `/logout` 命令的登出流程。包括刷新遥测数据、移除API密钥、清除安全存储、清理所有认证相关缓存等操作。支持可选的清除onboarding状态。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import { clearTrustedDeviceTokenCache } from '../../bridge/trustedDevice.js'
import { Text } from '../../ink.js'
import { refreshGrowthBookAfterAuthChange } from '../../services/analytics/growthbook.js'
import { getGroveNoticeConfig, getGroveSettings } from '../../services/api/grove.js'
import { clearPolicyLimitsCache } from '../../services/policyLimits/index.js'
import { clearRemoteManagedSettingsCache } from '../../services/remoteManagedSettings/index.js'
import { getClaudeAIOAuthTokens, removeApiKey } from '../../utils/auth.js'
import { clearBetasCaches } from '../../utils/betas.js'
import { saveGlobalConfig } from '../../utils/config.js'
import { gracefulShutdownSync } from '../../utils/gracefulShutdown.js'
import { getSecureStorage } from '../../utils/secureStorage/index.js'
import { clearToolSchemaCache } from '../../utils/toolSchemaCache.js'
import { resetUserCache } from '../../utils/user.js'
```

### 主要函数

#### performLogout(options): Promise<void>

- **类型**: `async function`
- **参数**:
  - `clearOnboarding`: `boolean` - 是否清除onboarding状态，默认 `false`
- **用途**: 执行登出操作的核心逻辑

**执行流程**:

1. 延迟加载并刷新遥测数据 (`flushTelemetry`)
2. 移除API密钥 (`removeApiKey`)
3. 删除安全存储中的所有数据
4. 清理认证相关缓存 (`clearAuthRelatedCaches`)
5. 更新全局配置：
   - 如果 `clearOnboarding` 为true，重置onboarding相关字段
   - 清除 `oauthAccount`

#### clearAuthRelatedCaches(): Promise<void>

- **类型**: `async function`
- **用途**: 清理所有与认证相关的缓存

**清理的缓存**:
- OAuth令牌缓存 (`getClaudeAIOAuthTokens.cache`)
- 可信设备令牌缓存 (`clearTrustedDeviceTokenCache`)
- Beta功能缓存 (`clearBetasCaches`)
- 工具模式缓存 (`clearToolSchemaCache`)
- 用户数据缓存 (`resetUserCache`)
- GrowthBook状态 (`refreshGrowthBookAfterAuthChange`)
- Grove配置缓存 (`getGroveNoticeConfig.cache`, `getGroveSettings.cache`)
- 远程管理设置缓存 (`clearRemoteManagedSettingsCache`)
- 策略限制缓存 (`clearPolicyLimitsCache`)

#### call(): Promise<React.ReactNode>

- **类型**: `async function`
- **返回值**: `Promise<React.ReactNode>`
- **用途**: 命令入口函数

**执行流程**:

1. 调用 `performLogout({ clearOnboarding: true })`
2. 显示成功消息
3. 200毫秒后执行优雅关闭 (`gracefulShutdownSync`)

---

## 设计要点

1. **遥测优先**: 在清除凭证之前先刷新遥测数据，防止组织数据泄漏。

2. **懒加载遥测**: `flushTelemetry` 使用动态导入，避免启动时加载约1.1MB的OpenTelemetry代码。

3. **安全存储清理**: 完全删除安全存储中的数据，不只是API密钥。

4. **延迟关闭**: 给用户200毫秒时间看到成功消息后再关闭。

---

## 与其他文件的关系

**依赖**:
- `clearTrustedDeviceTokenCache` (`../../bridge/trustedDevice.js`)
- `refreshGrowthBookAfterAuthChange` (`../../services/analytics/growthbook.js`)
- `getGroveNoticeConfig`, `getGroveSettings` (`../../services/api/grove.js`)
- `clearPolicyLimitsCache` (`../../services/policyLimits/index.js`)
- `clearRemoteManagedSettingsCache` (`../../services/remoteManagedSettings/index.js`)
- `getClaudeAIOAuthTokens`, `removeApiKey` (`../../utils/auth.js`)
- `getSecureStorage` (`../../utils/secureStorage/index.js`)
- `gracefulShutdownSync` (`../../utils/gracefulShutdown.js`)

**被依赖**:
- `logout/index.ts` - 导出为命令配置

---

## 注意事项

1. **不可逆操作**: 登出后所有本地凭证和缓存都将被清除。

2. **Onboarding重置**: 传入 `clearOnboarding: true` 会将用户状态恢复到首次使用状态。

3. **缓存失效**: 所有memoized的认证相关数据都会被清理。
