# setupGitHubActions.ts — GitHub Actions设置逻辑

> **一句话总结**：实现GitHub Actions工作流创建和API密钥配置的核心逻辑。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/setupGitHubActions.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 325 行 |
| 主要职责 | 创建分支、工作流文件、设置密钥、打开PR页面 |

---

## 功能概述

该文件实现了GitHub Actions设置的核心逻辑。包括创建Git分支、创建工作流文件、设置API密钥为仓库密钥、打开预填充的PR页面等操作。完整的分析追踪和错误处理。

---

## 核心内容详解

### 导入与依赖

```typescript
import { type AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS, logEvent } from 'src/services/analytics/index.js'
import { saveGlobalConfig } from 'src/utils/config.js'
import { CODE_REVIEW_PLUGIN_WORKFLOW_CONTENT, PR_BODY, PR_TITLE, WORKFLOW_CONTENT } from '../../constants/github-app.js'
import { openBrowser } from '../../utils/browser.js'
import { execFileNoThrow } from '../../utils/execFileNoThrow.js'
import { logError } from '../../utils/log.js'
import type { Workflow } from './types.js'
```

### 主要函数

#### createWorkflowFile(...): Promise<void>

- **类型**: `async function`
- **用途**: 创建或更新单个工作流文件

**参数**:
- `repoName`: 仓库名称
- `branchName`: 分支名称
- `workflowPath`: 工作流文件路径
- `workflowContent`: 工作流内容
- `secretName`: 密钥名称
- `message`: 提交消息
- `context?`: 上下文信息

**逻辑**:

1. 检查工作流文件是否已存在，获取SHA（用于更新）
2. 根据密钥名称替换工作流内容中的密钥引用
3. 使用GitHub API创建或更新文件
4. 错误处理：422错误表示文件已存在

#### setupGitHubActions(...): Promise<void>

- **类型**: `async function`
- **用途**: 执行完整的GitHub Actions设置流程

**参数**:
- `repoName`: 仓库名称
- `apiKeyOrOAuthToken`: API密钥或OAuth令牌
- `secretName`: 密钥名称
- `updateProgress`: 进度更新回调
- `skipWorkflow`: 是否跳过工作流创建
- `selectedWorkflows`: 选择的工作流列表
- `authType`: 认证类型（api_key 或 oauth_token）
- `context?`: 上下文信息

**执行流程**:

1. 记录开始事件（分析追踪）
2. 检查仓库是否存在
3. 获取默认分支
4. 获取分支SHA
5. 如果不跳过工作流：
   - 创建新分支
   - 创建选定的工作流文件（claude.yml、claude-code-review.yml）
6. 设置API密钥为仓库密钥
7. 打开预填充的PR页面
8. 记录完成事件
9. 更新全局配置的 `githubActionSetupCount`

---

## 设计要点

1. **分析追踪**: 详细记录设置过程的开始、失败和完成事件。

2. **错误分类**: 根据错误类型提供不同的帮助信息。

3. **工作流模板**: 从常量导入工作流内容，支持多个工作流。

4. **密钥替换**: 根据密钥名称动态替换工作流中的密钥引用。

5. **浏览器打开**: 使用 `openBrowser` 打开预填充的PR创建页面。

---

## 与其他文件的关系

**依赖**:
- `logEvent` (`src/services/analytics/index.js`)
- `saveGlobalConfig` (`src/utils/config.js`)
- `WORKFLOW_CONTENT`, `CODE_REVIEW_PLUGIN_WORKFLOW_CONTENT` (`../../constants/github-app.js`)
- `openBrowser` (`../../utils/browser.js`)
- `execFileNoThrow` (`../../utils/execFileNoThrow.js`)
- `Workflow` 类型 (`./types.js`)

**被依赖**:
- `install-github-app.tsx` - 调用设置逻辑

---

## 注意事项

1. **权限要求**: 需要 `repo` 和 `workflow` 权限才能创建分支和工作流。

2. **422错误**: 工作流文件已存在时返回特定错误信息。

3. **PR预填充**: PR标题和正文通过URL参数预填充。

4. **计数器**: 成功设置后递增 `githubActionSetupCount`。

5. **密钥名称**:
   - `CLAUDE_CODE_OAUTH_TOKEN`: 使用 `claude_code_oauth_token` 参数
   - 其他名称: 使用 `anthropic_api_key` 参数并替换密钥引用
