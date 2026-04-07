# install-github-app.tsx — GitHub App安装主流程

> **一句话总结**：实现完整的Claude GitHub Actions安装向导流程。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/install-github-app.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 约600行（编译后） |
| 主要职责 | 协调多步骤安装流程，管理状态转换 |

---

## 功能概述

该文件实现了 `/install-github-app` 命令的完整安装流程。包含多个步骤：检查GitHub CLI、选择仓库、配置API密钥、处理现有工作流、创建分支和工作流文件、设置密钥、打开PR页面等。完整的错误处理和用户引导。

---

## 核心内容详解

### 导入与依赖

```typescript
import { execa } from 'execa'
import React, { useCallback, useState } from 'react'
import { type AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS, logEvent } from 'src/services/analytics/index.js'
import { WorkflowMultiselectDialog } from '../../components/WorkflowMultiselectDialog.js'
import { GITHUB_ACTION_SETUP_DOCS_URL } from '../../constants/github-app.js'
import { useExitOnCtrlCDWithKeybindings } from '../../hooks/useExitOnCtrlCDWithKeybindings.js'
import type { KeyboardEvent } from '../../ink/events/keyboard-event.js'
import { Box } from '../../ink.js'
import type { LocalJSXCommandOnDone } from '../../types/command.js'
import { getAnthropicApiKey, isAnthropicAuthEnabled } from '../../utils/auth.js'
import { openBrowser } from '../../utils/browser.js'
import { execFileNoThrow } from '../../utils/execFileNoThrow.js'
import { getGithubRepo } from '../../utils/git.js'
import { plural } from '../../utils/stringUtils.js'
import { ApiKeyStep } from './ApiKeyStep.js'
import { CheckExistingSecretStep } from './CheckExistingSecretStep.js'
import { CheckGitHubStep } from './CheckGitHubStep.js'
import { ChooseRepoStep } from './ChooseRepoStep.js'
import { CreatingStep } from './CreatingStep.js'
import { ErrorStep } from './ErrorStep.js'
import { ExistingWorkflowStep } from './ExistingWorkflowStep.js'
import { InstallAppStep } from './InstallAppStep.js'
import { OAuthFlowStep } from './OAuthFlowStep.js'
import { SuccessStep } from './SuccessStep.js'
import { setupGitHubActions } from './setupGitHubActions.js'
import type { State, Warning, Workflow } from './types.js'
import { WarningsStep } from './WarningsStep.js'
```

### 状态定义

```typescript
const INITIAL_STATE: State = {
  step: 'check-gh',
  selectedRepoName: '',
  currentRepo: '',
  useCurrentRepo: false,
  apiKeyOrOAuthToken: '',
  useExistingKey: true,
  currentWorkflowInstallStep: 0,
  warnings: [],
  secretExists: false,
  secretName: 'ANTHROPIC_API_KEY',
  useExistingSecret: true,
  workflowExists: false,
  selectedWorkflows: ['claude', 'claude-review'] as Workflow[],
  selectedApiKeyOption: 'existing' as 'existing' | 'new' | 'oauth',
  authType: 'api_key'
}
```

### 主要组件

#### InstallGitHubApp(props): React.ReactNode

- **类型**: React函数组件
- **用途**: 管理安装流程的状态和步骤转换

**主要步骤**:

1. **check-gh**: 检查GitHub CLI安装和认证
2. **warnings**: 显示检查警告（如果有）
3. **choose-repo**: 选择目标仓库
4. **check-existing-secret**: 检查现有密钥
5. **api-key**: 配置API密钥（现有、新密钥或OAuth）
6. **oauth-flow**: OAuth令牌生成流程
7. **existing-workflow**: 处理现有工作流
8. **select-workflows**: 选择要安装的工作流
9. **creating**: 创建分支、工作流、设置密钥
10. **install-app**: 引导安装GitHub App
11. **success**: 显示成功信息
12. **error**: 显示错误信息

#### checkGitHubCLI(): Promise<void>

- **用途**: 检查GitHub CLI状态

**检查项**:
- gh是否已安装
- gh是否已认证
- 是否有repo和workflow权限

#### 流程控制

- 根据当前状态渲染对应的步骤组件
- 状态转换通过 `setState` 和回调函数处理
- 错误处理统一转到error状态

---

## 设计要点

1. **多步骤向导**: 将复杂流程分解为多个可管理的步骤。

2. **状态管理**: 使用React state管理整个流程的状态。

3. **错误恢复**: 在关键步骤提供重试和退出选项。

4. **分析追踪**: 详细记录每个步骤的开始和完成事件。

5. **灵活性**: 支持多种配置方式（现有密钥、新密钥、OAuth）。

6. **现有资源处理**: 智能处理已存在的密钥和工作流。

---

## 与其他文件的关系

**依赖**:
- `ApiKeyStep`, `ChooseRepoStep`, `CreatingStep`, `ErrorStep`, `ExistingWorkflowStep`, `InstallAppStep`, `OAuthFlowStep`, `SuccessStep`, `WarningsStep`, `CheckGitHubStep`, `CheckExistingSecretStep`（各步骤组件）
- `setupGitHubActions` (`./setupGitHubActions.js`)
- `State`, `Warning`, `Workflow` 类型 (`./types.js`)
- `WorkflowMultiselectDialog` (`../../components/WorkflowMultiselectDialog.js`)

**被依赖**:
- `install-github-app/index.ts` - 导出为命令配置

---

## 注意事项

1. **GitHub CLI依赖**: 整个流程依赖GitHub CLI (gh) 命令。

2. **权限检查**: 检查repo和workflow权限，缺失时显示警告。

3. **OAuth支持**: 如果用户已登录Anthropic账户，支持OAuth流程。

4. **工作流选择**: 支持同时安装claude和claude-review两个工作流。

5. **PR创建**: 最终打开预填充的PR页面，用户需要手动创建和合并PR。
