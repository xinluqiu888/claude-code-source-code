# OAuthFlowStep.tsx — OAuth令牌生成步骤

> **一句话总结**：实现OAuth登录流程，生成用于GitHub Actions的访问令牌。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/OAuthFlowStep.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 约80行（编译后） |
| 主要职责 | 执行OAuth登录并获取访问令牌 |

---

## 功能概述

该组件实现了GitHub App安装流程中的OAuth令牌生成步骤。通过OAuth流程让用户授权，生成可用于GitHub Actions的访问令牌，替代API密钥使用。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import { useCallback, useEffect, useRef, useState } from 'react'
import { type AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS, logEvent } from 'src/services/analytics/index.js'
import { KeyboardShortcutHint } from '../../components/design-system/KeyboardShortcutHint.js'
import { Spinner } from '../../components/Spinner.js'
import TextInput from '../../components/TextInput.js'
import { useTerminalSize } from '../../hooks/useTerminalSize.js'
import type { KeyboardEvent } from '../../ink/events/keyboard-event.js'
import { setClipboard } from '../../ink/termio/osc.js'
import { Box, Link, Text } from '../../ink.js'
import { OAuthService } from '../../services/oauth/index.js'
import { saveOAuthTokensIfNeeded } from '../../utils/auth.js'
import { logError } from '../../utils/log.js'
```

### 接口定义

```typescript
interface OAuthFlowStepProps {
  onSuccess: (token: string) => void
  onCancel: () => void
}

type OAuthStatus =
  | { state: 'starting' }
  | { state: 'waiting_for_login'; url: string }
  | { state: 'processing' }
  | { state: 'success'; token: string }
  | { state: 'error'; message: string; toRetry?: OAuthStatus }
  | { state: 'about_to_retry'; nextState: OAuthStatus }
```

### 主要组件

#### OAuthFlowStep(props): React.ReactNode

- **类型**: React函数组件
- **用途**: 执行OAuth流程并获取令牌

**流程状态**:

1. **starting**: 初始化OAuth服务
2. **waiting_for_login**: 等待用户在浏览器中登录
3. **processing**: 处理授权码
4. **success**: 成功获取令牌
5. **error**: 流程出错，支持重试

**代码输入**:

- 支持用户粘贴从浏览器回调URL复制的授权码
- 格式: `authorizationCode#state`

---

## 设计要点

1. **状态机**: 使用状态机管理OAuth流程的各个阶段。

2. **URL复制**: 提供将授权URL复制到剪贴板的功能。

3. **重试机制**: 支持错误后重试，保留之前的状态。

4. **令牌保存**: 成功获取令牌后自动保存到认证系统。

---

## 与其他文件的关系

**依赖**:
- `OAuthService` (`../../services/oauth/index.js`)
- `saveOAuthTokensIfNeeded` (`../../utils/auth.js`)
- `Spinner` (`../../components/Spinner.js`)
- `TextInput` (`../../components/TextInput.js`)

**被依赖**:
- `install-github-app.tsx` - 作为主流程的步骤组件使用

---

## 注意事项

1. **浏览器登录**: 用户需要在浏览器中完成授权流程。

2. **代码格式**: 期望的授权码格式为 `code#state`。

3. **剪贴板集成**: 支持将授权URL复制到系统剪贴板。

4. **错误处理**: 提供详细的错误信息和重试选项。
