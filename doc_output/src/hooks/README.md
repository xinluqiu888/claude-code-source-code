# hooks/ — React Hooks 目录

> **一句话总结**：Claude Code 的 React Hooks 集合，涵盖输入处理、状态管理、通知、权限等多种功能

---

## 目录职责

本目录是 Claude Code 的核心 Hooks 集合，提供：
1. **输入处理**: 文件建议、文本输入、Vim 输入、搜索输入
2. **状态管理**: 设置变更、技能变更、会话管理
3. **通知系统**: 各类通知 Hooks（见 notifs/ 子目录）
4. **权限管理**: 工具权限上下文和处理器（见 toolPermission/ 子目录）
5. **IDE 集成**: IDE 连接、状态、选择
6. **任务管理**: 后台任务导航、任务列表监听
7. **远程会话**: 直连、远程会话、SSH 会话
8. **工具建议**: 文件建议、统一建议

## 子目录

| 目录 | 职责简述 |
|------|---------|
| [notifs/](./notifs/README.md) | 通知相关 Hooks |
| [toolPermission/](./toolPermission/README.md) | 工具权限管理 |

## 核心文件分类

### 输入和搜索

| 文件 | 职责 |
|------|------|
| fileSuggestions.ts | 文件路径建议和索引管理 |
| unifiedSuggestions.ts | 统一建议（文件 + MCP + Agent） |
| renderPlaceholder.ts | 输入框占位符渲染 |
| useTextInput.ts | 文本输入管理 |
| useVimInput.ts | Vim 模式输入 |
| useSearchInput.ts | 搜索输入管理 |
| useTypeahead.tsx | 类型提示 |

### 状态和配置

| 文件 | 职责 |
|------|------|
| useSettings.ts | 设置管理 |
| useSettingsChange.ts | 设置变更监听 |
| useSkillsChange.ts | 技能变更监听 |
| useDynamicConfig.ts | 动态配置 |
| useSessionBackgrounding.ts | 会话后台化 |

### IDE 和 LSP

| 文件 | 职责 |
|------|------|
| useIdeConnectionStatus.ts | IDE 连接状态 |
| useIdeSelection.ts | IDE 选择管理 |
| useIdeLogging.ts | IDE 日志记录 |
| useLspPluginRecommendation.tsx | LSP 插件推荐 |

### 权限和工具

| 文件 | 职责 |
|------|------|
| useCanUseTool.tsx | 工具使用权限检查 |
| useMergedTools.ts | 合并工具列表 |
| toolPermission/ | 权限上下文和处理器 |

### 通知（部分示例）

| 文件 | 职责 |
|------|------|
| useUpdateNotification.ts | 更新可用通知 |
| useNotifyAfterTimeout.ts | 超时通知 |
| useBlink.ts | 闪烁动画 |

## 设计原则

### 1. 单一职责

每个 Hook 负责单一功能，如 `useBlink` 只处理闪烁动画。

### 2. 可组合性

Hooks 可以组合使用，如 `useStartupNotification` 被多个通知 Hook 使用。

### 3. 远程模式保护

涉及本地功能的 Hook 通常检查 `getIsRemoteMode()`。

### 4. 特性标志

新功能通过 `feature('XXX')` 控制，便于灰度发布。

### 5. 性能优化

使用 `useMemo`、`useCallback` 和 React Compiler 优化渲染。
