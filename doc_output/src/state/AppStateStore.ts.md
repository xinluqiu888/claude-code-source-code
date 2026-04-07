# AppStateStore.ts — 应用程序状态管理中心

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/state/AppStateStore.ts`
- **类型**: TypeScript 模块
- **导出内容**: `AppState` 类型、`AppStateStore` 类型、`getDefaultAppState()` 函数
- **依赖关系**: 
  - 导入: `notifications.js`, `todo/types.js`, `bridgePermissionCallbacks.js`, `commands.js`, `mcp/types.js`, `PromptSuggestion.js`, `Tool.js`, `tasks/types.js`, `agentColorManager.js`, `loadAgentsDir.js`, `ExitPlanModeV2Tool.js`, `ids.js`, `message.js`, `plugin.js`, `utils.js`, `commitAttribution.js`, `effort.js`, `fileHistory.js`, `postSamplingHooks.js`, `sessionHooks.js`, `model.js`, `denialTracking.js`, `PermissionMode.js`, `settings.js`, `types.js`, `thinking.js`, `store.js`

## 功能概述

本文件是 Claude Code 应用程序的核心状态定义文件，定义了 `AppState` 类型——一个包含 400+ 行代码的巨型状态类型，涵盖了应用程序运行时的所有状态数据。该状态管理包括：任务管理、插件系统、MCP 集成、权限控制、UI 状态、远程会话、桥接连接、团队/代理协作等各个方面。

## 核心内容详解

### 1. CompletionBoundary 类型 (第41-51行)

定义了各种操作完成边界的类型，用于标记会话中的关键节点：
- `complete`: 正常完成，包含完成时间和输出token数
- `bash`: Bash命令执行，包含命令字符串和完成时间
- `edit`: 编辑操作，包含工具名、文件路径和完成时间
- `denied_tool`: 被拒绝的工具调用

### 2. SpeculationResult & SpeculationState 类型 (第52-79行)

推测执行相关类型：
- `SpeculationResult`: 包含消息、边界和时间节省
- `SpeculationState`: 可以是 `idle` 或 `active`，活跃状态包含中止函数、启动时间、消息引用、边界等信息

### 3. FooterItem 类型 (第81-88行)

底部栏可选项目：
- `'tasks'`, `'tmux'`, `'bagel'`, `'teams'`, `'bridge'`, `'companion'`

### 4. AppState 类型 (第89-452行)

这是核心状态类型，使用 `DeepImmutable` 包装以确保不可变性。主要包含：

**设置与配置**:
- `settings`: 用户设置 (SettingsJson)
- `verbose`: 详细日志模式
- `mainLoopModel`: 主循环使用的模型
- `toolPermissionContext`: 工具权限上下文

**UI 状态**:
- `statusLineText`: 状态栏文本
- `expandedView`: 扩展视图模式
- `viewSelectionMode`: 视图选择模式
- `footerSelection`: 底部栏选中项

**任务与代理**:
- `tasks`: 任务状态字典
- `agentNameRegistry`: 代理名称注册表
- `viewingAgentTaskId`: 当前查看的代理任务ID
- `foregroundedTaskId`: 前台任务ID

**远程与桥接**:
- `remoteSessionUrl`, `remoteConnectionStatus`: 远程会话
- `replBridgeEnabled`, `replBridgeConnected` 等: REPL 桥接状态

**插件与MCP**:
- `mcp`: MCP 客户端、工具、命令、资源
- `plugins`: 启用的插件、禁用的插件、命令、错误

**高级功能**:
- `speculation`: 推测执行状态
- `computerUseMcpState`: 计算机使用 MCP 状态
- `teamContext`: 团队/代理群上下文
- `inbox`: 消息收件箱
- `todos`: 待办事项

### 5. AppStateStore 类型 (第454行)

AppState 的 Store 类型包装，提供 `getState`, `setState`, `subscribe` 方法。

### 6. getDefaultAppState() 函数 (第456-569行)

创建默认应用程序状态的工厂函数：
- 设置初始权限模式（根据是否是队友且需要plan模式）
- 初始化空任务、代理注册表
- 设置默认的 MCP 和插件状态
- 配置默认的文件历史和归属状态
- 设置提示建议和推测执行的初始状态

## 设计要点

1. **单一事实来源**: AppState 是应用程序的单一状态源，所有组件都从中读取状态
2. **不可变性**: 使用 `DeepImmutable` 确保状态不可变，通过 `setState` 更新
3. **模块化设计**: 相关状态分组（如 `mcp`, `plugins`, `computerUseMcpState`）
4. **条件特性**: 使用可选字段 (`?`) 表示条件性功能（如 `showTeammateMessagePreview`）
5. **性能考虑**: `tasks` 被排除在 `DeepImmutable` 外，因为包含函数类型

## 与其他文件的关系

- **store.ts**: 提供底层 Store 实现
- **onChangeAppState.ts**: 监听状态变化并执行副作用
- **selectors.ts**: 从 AppState 派生计算状态
- **teammateViewHelpers.ts**: 操作与队友视图相关的状态

## 注意事项

1. 该文件包含大量导入依赖，注意避免循环依赖
2. 状态非常庞大，更新时要注意性能影响
3. `isUltraplanMode` 等特殊标志有特定的生命周期管理
4. 使用懒加载 require 避免循环依赖（如第459-462行）
