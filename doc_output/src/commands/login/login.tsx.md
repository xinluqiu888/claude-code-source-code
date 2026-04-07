# login.tsx — 用户登录流程

> **一句话总结**：实现OAuth登录流程，处理登录成功后的状态刷新和缓存清理。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/login/login.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 103 行 |
| 主要职责 | 实现交互式登录界面和登录后状态管理 |

---

## 功能概述

该文件实现了 `/login` 命令的交互式登录流程。使用 `ConsoleOAuthFlow` 组件处理OAuth认证，登录成功后执行一系列状态刷新操作，包括清除缓存、刷新GrowthBook特性开关、重新注册可信设备等。

---

## 核心内容详解

### 导入与依赖

```typescript
import { feature } from 'bun:bundle'
import * as React from 'react'
import { resetCostState } from '../../bootstrap/state.js'
import { clearTrustedDeviceToken, enrollTrustedDevice } from '../../bridge/trustedDevice.js'
import type { LocalJSXCommandContext } from '../../commands.js'
import { ConfigurableShortcutHint } from '../../components/ConfigurableShortcutHint.js'
import { ConsoleOAuthFlow } from '../../components/ConsoleOAuthFlow.js'
import { Dialog } from '../../components/design-system/Dialog.js'
import { useMainLoopModel } from '../../hooks/useMainLoopModel.js'
import { Text } from '../../ink.js'
import { refreshGrowthBookAfterAuthChange } from '../../services/analytics/growthbook.js'
import { refreshPolicyLimits } from '../../services/policyLimits/index.js'
import { refreshRemoteManagedSettings } from '../../services/remoteManagedSettings/index.js'
import type { LocalJSXCommandOnDone } from '../../types/command.js'
import { stripSignatureBlocks } from '../../utils/messages.js'
import { checkAndDisableAutoModeIfNeeded, checkAndDisableBypassPermissionsIfNeeded, resetAutoModeGateCheck, resetBypassPermissionsCheck } from '../../utils/permissions/bypassPermissionsKillswitch.js'
import { resetUserCache } from '../../utils/user.js'
```

### 主要函数

#### call(onDone, context): Promise<React.ReactNode>

- **类型**: `async function`
- **返回值**: `Promise<React.ReactNode>`
- **用途**: 主入口函数，启动登录流程

**执行流程**:

1. 渲染 `Login` 组件
2. 登录成功后执行以下操作：
   - 调用 `context.onChangeAPIKey()` 通知API密钥变更
   - 使用 `stripSignatureBlocks` 清理消息中的签名块
   - 重置成本状态 (`resetCostState`)
   - 刷新远程管理设置 (`refreshRemoteManagedSettings`)
   - 刷新策略限制 (`refreshPolicyLimits`)
   - 重置用户缓存 (`resetUserCache`)
   - 刷新GrowthBook特性开关 (`refreshGrowthBookAfterAuthChange`)
   - 清除并重新注册可信设备 (`clearTrustedDeviceToken`, `enrollTrustedDevice`)
   - 重置权限绕过检查 (`resetBypassPermissionsCheck`, `checkAndDisableBypassPermissionsIfNeeded`)
   - 如果启用了 `TRANSCRIPT_CLASSIFIER` 特性，重置自动模式检查
   - 递增 `authVersion` 触发依赖认证数据的钩子重新获取

#### Login(props): React.ReactNode

- **类型**: React函数组件
- **参数**:
  - `onDone`: `(success: boolean, mainLoopModel: string) => void` - 登录完成回调
  - `startingMessage?`: `string` - 可选的起始消息
- **用途**: 渲染登录对话框

---

## 设计要点

1. **登录后刷新**: 保持与 `src/interactiveHelpers.tsx` 中的 onboarding 逻辑同步。

2. **签名块清理**: 移除绑定到旧API密钥的签名块（thinking、connector_text），防止新密钥拒绝旧签名。

3. **可信设备管理**: 清除旧账户的可信设备令牌，重新注册为新账户的可信设备。

4. **特性开关检查**: 根据 `TRANSCRIPT_CLASSIFIER` 特性开关决定是否重置自动模式检查。

---

## 与其他文件的关系

**依赖**:
- `resetCostState` (`../../bootstrap/state.js`)
- `clearTrustedDeviceToken`, `enrollTrustedDevice` (`../../bridge/trustedDevice.js`)
- `ConsoleOAuthFlow` (`../../components/ConsoleOAuthFlow.js`)
- `Dialog` (`../../components/design-system/Dialog.js`)
- `refreshGrowthBookAfterAuthChange` (`../../services/analytics/growthbook.js`)
- `refreshPolicyLimits` (`../../services/policyLimits/index.js`)
- `refreshRemoteManagedSettings` (`../../services/remoteManagedSettings/index.js`)

**被依赖**:
- `login/index.ts` - 导出为命令配置

---

## 注意事项

1. **同步维护**: 登录后的刷新逻辑需要与 onboarding 流程保持同步。

2. **缓存清理**: 用户数据缓存在GrowthBook刷新之前清理，确保获取到最新的凭证信息。

3. **10分钟窗口**: 可信设备注册在10分钟的新鲜会话窗口内有效。
