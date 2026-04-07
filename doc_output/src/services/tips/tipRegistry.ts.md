# tipRegistry.ts — 提示注册表

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/tips/tipRegistry.ts`
- **作用域**: 提示定义、过滤和相关性判断
- **主要导出**:
  - `getRelevantTips`: 获取与当前上下文相关的提示
  - `externalTips`: 外部用户提示数组（供内部使用）

## 功能概述

定义所有旋转提示的内容和显示条件。包含 40+ 个内置提示，涵盖功能介绍、使用技巧、插件推荐等。根据用户配置、平台、使用历史等条件过滤和排序。

## 核心内容详解

### 类型定义（推断）

```typescript
type Tip = {
  id: string
  content: (ctx?: TipContext) => Promise<string>
  cooldownSessions: number
  isRelevant?: (ctx?: TipContext) => Promise<boolean>
}

type TipContext = {
  bashTools?: Set<string>
  readFileState?: FileStateCache
  theme?: string
}
```

### 内置提示分类

#### 新用户提示
- `new-user-warmup`: 新用户热身建议（冷却3会话）
- `plan-mode-for-complex-tasks`: 复杂任务使用计划模式（冷却5会话）

#### Git 和开发工作流
- `git-worktrees`: Git 工作树功能
- `color-when-multi-clauding`: 多会话颜色区分

#### 终端集成
- `terminal-setup`: 终端设置
- `shift-enter`: 多行消息发送
- `shift-enter-setup`: Shift+Enter 设置

#### 命令和功能介绍
- `memory-command`: /memory 命令
- `theme-command`: /theme 命令
- `prompt-queue`: 提示队列功能
- `todo-list`: 待办列表功能
- `permissions`: 权限预批准

#### IDE 集成
- `vscode-command-install`: VS Code 命令安装
- `ide-upsell-external-terminal`: IDE 连接

#### 应用集成
- `install-github-app`: GitHub 应用安装
- `install-slack-app`: Slack 应用安装

#### 进阶功能
- `custom-commands`: 自定义命令/.claude/skills
- `custom-agents`: 自定义代理
- `agent-flag`: --agent 参数
- `desktop-app`: 桌面应用
- `desktop-shortcut`: 桌面快捷方式
- `web-app`: Web 应用
- `mobile-app`: 移动应用

#### 插件推荐
- `frontend-design-plugin`: 前端设计插件
- `vercel-plugin`: Vercel 插件

#### 1P 用户功能
- `effort-high-nudge`: 高 effort 模式建议
- `subagent-fanout-nudge`: 子代理分发建议
- `loop-command-nudge`: /loop 命令建议

#### 奖励和推广
- `guest-passes`: 客座通行证
- `overage-credit`: 超额信用额度

#### 内部员工提示 (USER_TYPE === 'ant')
- `important-claudemd`: CLAUDE.md 重要规则
- `skillify`: /skillify 命令

### 核心函数

#### `getRelevantTips(context?)`
获取与当前上下文相关的提示：

**流程**:
1. 获取自定义提示（来自设置覆盖）
2. 如果 `excludeDefault` 为 true 且有自定义提示，仅返回自定义提示
3. 否则，过滤内置提示：
   - 调用每个提示的 `isRelevant` 函数
   - 检查冷却会话数（`sessionsSinceLastShown >= cooldownSessions`）
4. 合并过滤后的内置提示和自定义提示

#### `isMarketplacePluginRelevant(pluginName, context, signals)`
判断插件推荐是否相关：

**信号检查**:
- `cli`: 检查是否使用了特定 CLI 命令
- `filePath`: 检查是否读取了特定类型文件

#### `isOfficialMarketplaceInstalled()`
检查官方市场插件是否已安装（带缓存）。

### 自定义提示

```typescript
function getCustomTips(): Tip[]
```

从设置中加载自定义提示覆盖：
- `spinnerTipsOverride.tips`: 自定义提示内容数组
- `spinnerTipsOverride.excludeDefault`: 是否排除默认提示

### 提示属性

**冷却会话数 (cooldownSessions)**:
- 0: 无冷却（如 vscode-command-install）
- 2-5: 短冷却（新功能推广）
- 10-15: 中等冷却（常用功能）
- 20-30: 长冷却（主题、颜色等）

**isRelevant 条件示例**:
```typescript
// 平台检查
isRelevant: async () => getPlatform() === 'macos'

// 使用历史检查
isRelevant: async () => config.numStartups > 10

// 功能启用检查
isRelevant: async () => !fileHistoryEnabled()

// 复合条件
isRelevant: async () => {
  const config = getGlobalConfig()
  const daysSinceLastUse = config.lastPlanModeUse
    ? (Date.now() - config.lastPlanModeUse) / (1000 * 60 * 60 * 24)
    : Infinity
  return hasOpusPlanMode && daysSinceLastUse > 3
}
```

## 设计要点

1. **条件显示**: 每个提示可根据用户环境、使用历史等条件决定是否显示
2. **冷却机制**: 防止频繁显示同一提示
3. **优先级排序**: 通过 `selectTipWithLongestTimeSinceShown` 实现公平的轮转
4. **自定义支持**: 允许通过设置注入自定义提示
5. **内部/外部分离**: 内部员工专用提示通过 `USER_TYPE` 环境变量控制

## 与其他文件的关系

- **tipScheduler.ts**: 调用 `getRelevantTips`
- **tipHistory.ts**: 调用 `getSessionsSinceLastShown`
- **utils/config.ts**: 获取全局配置
- **utils/settings/settings.ts**: 获取用户设置
- **services/analytics/growthbook.ts**: 获取特性开关值

## 注意事项

1. **异步 content**: 提示内容支持异步生成，可包含动态数据
2. **主题支持**: 部分提示接收 `theme` 参数用于颜色格式化
3. **缓存机制**: `isOfficialMarketplaceInstalled` 有模块级缓存
4. **错误处理**: 使用 try-catch 包装 `isRelevant` 函数防止崩溃
