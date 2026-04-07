# remoteSession.ts — 后台远程会话类型定义与资格检查

> **一句话总结**：定义后台远程会话的数据结构，并检查用户是否满足创建远程会话的所有前置条件。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/background/remote/remoteSession.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 98 |
| 主要职责 | 定义 `BackgroundRemoteSession` 类型及前置条件枚举，并实现 `checkBackgroundRemoteSessionEligibility` 用于多项并行资格验证。 |

---

## 功能概述

本文件是后台远程会话（Background Remote Session，用于 Teleport/CCR 功能）的核心入口，负责两件事：

首先，它定义了描述一个后台远程会话的完整数据结构 `BackgroundRemoteSession`，包括会话 ID、命令文本、启动时间、状态枚举（starting/running/completed/failed/killed）、任务列表、标题、类型标识，以及完整的 SDK 消息日志。

其次，通过联合类型 `BackgroundRemoteSessionPrecondition` 枚举了所有可能的前置条件失败情况，并在 `checkBackgroundRemoteSessionEligibility` 中并行执行各项检查：策略允许、Claude.ai 登录状态、远程环境可用性、Git 仓库存在、GitHub remote 可用及 GitHub App 安装状态。特别地，当 bundle seed 功能门控开启时，只需要本地存在 `.git/` 目录即可，无需 GitHub remote 和 App 检查。

---

## 核心内容详解

### 导入与依赖

- `SDKMessage`：来自 `agentSdkTypes.js`，表示 Agent SDK 消息类型。
- `checkGate_CACHED_OR_BLOCKING`：来自 growthbook，用于功能门控检查（`tengu_ccr_bundle_seed_enabled`）。
- `isPolicyAllowed`：策略限制检查，`allow_remote_sessions` 策略控制整体开关。
- `detectCurrentRepositoryWithHost`：检测当前 Git 仓库并返回带主机信息的仓库对象。
- `isEnvTruthy`：环境变量布尔值解析工具。
- `TodoList`：来自 todo 模块的任务列表类型。
- `preconditions.ts` 中的四个检查函数。

### 主要类/函数/接口

**`BackgroundRemoteSession`（类型）**
```typescript
type BackgroundRemoteSession = {
  id: string
  command: string
  startTime: number
  status: 'starting' | 'running' | 'completed' | 'failed' | 'killed'
  todoList: TodoList
  title: string
  type: 'remote_session'
  log: SDKMessage[]
}
```

**`BackgroundRemoteSessionPrecondition`（联合类型）**
表示六种可能的前置条件失败：未登录、无远程环境、非 Git 仓库、无 Git remote、GitHub App 未安装、策略阻止。

**`checkBackgroundRemoteSessionEligibility`（异步函数）**
- 参数：`{ skipBundle?: boolean }`，可选跳过 bundle seed 逻辑。
- 返回：`BackgroundRemoteSessionPrecondition[]`，空数组表示所有检查通过。
- 优先检查策略，若策略阻止则提前返回。
- 其余检查使用 `Promise.all` 并行执行。

### 数据流与逻辑流程

```
checkBackgroundRemoteSessionEligibility()
  ├── isPolicyAllowed('allow_remote_sessions') — 策略检查（最高优先级）
  ├── Promise.all([
  │     checkNeedsClaudeAiLogin(),        — 登录检查
  │     checkHasRemoteEnvironment(),      — 远程环境检查
  │     detectCurrentRepositoryWithHost() — 仓库检测
  │   ])
  ├── checkIsInGitRepo()                  — 本地 .git/ 检查（同步）
  ├── bundleSeedGateOn?
  │     是 → 跳过 remote/app 检查
  │     否 → checkGithubAppInstalled()    — GitHub App 安装检查
  └── 返回 errors[]
```

### 对外接口（导出内容）

- `BackgroundRemoteSession`：类型导出
- `BackgroundRemoteSessionPrecondition`：类型导出
- `checkBackgroundRemoteSessionEligibility`：函数导出

---

## 设计要点

- **策略优先**：策略检查（`isPolicyAllowed`）放在最前面，一旦被策略阻止立即返回，避免不必要的网络请求。
- **并行检查**：登录、远程环境和仓库检测通过 `Promise.all` 并行执行，提升性能。
- **Bundle Seed 快捷路径**：当 `CCR_FORCE_BUNDLE`、`CCR_ENABLE_BUNDLE` 或功能门控开启时，对于仅有本地 `.git/` 的仓库也允许使用远程功能，跳过 GitHub 相关检查。
- **错误累积模式**：收集所有失败的前置条件后一次性返回，而非快速失败，便于 UI 层一次展示所有问题。

---

## 与其他文件的关系

- **依赖**：
  - `src/utils/background/remote/preconditions.ts`：提供具体的前置条件检查函数
  - `src/services/analytics/growthbook.js`：功能门控检查
  - `src/services/policyLimits/index.js`：策略限制
  - `src/utils/detectRepository.js`：仓库检测
  - `src/utils/envUtils.js`：环境变量工具
  - `src/utils/todo/types.js`：任务列表类型

- **被依赖**：
  - UI 层的 Teleport 相关组件（如 `TeleportError.tsx`）使用此函数决定是否显示资格提示
  - 后台会话管理逻辑用于创建和跟踪远程会话

---

## 注意事项

- `checkIsInGitRepo()` 是同步函数（使用 `findGitRoot`），而其他检查都是异步的，所以它在 `Promise.all` 之外单独调用。
- Bundle seed 的判断依赖于 growthbook 的功能门控，可能有缓存延迟（使用 `CACHED_OR_BLOCKING` 变体，首次调用可能阻塞等待）。
- `skipBundle` 参数允许调用方在特定场景下强制走标准路径，不使用 bundle 快捷方式。
