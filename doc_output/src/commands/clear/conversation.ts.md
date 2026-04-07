# conversation.ts — 对话清除核心逻辑

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/clear/conversation.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 251 行 |
| 主要职责 | 实现完整的对话清除逻辑，包括消息清理、缓存清理、会话管理 |

## 功能概述

该文件实现了 `/clear` 命令的核心清除逻辑。它清理对话历史、各种缓存、重置会话状态，同时支持保留后台任务。这是 Claude Code 中重要的会话管理功能。

## 核心内容详解

### 主要函数

**clearConversation**
```typescript
export async function clearConversation({
  setMessages,
  readFileState,
  discoveredSkillNames,
  loadedNestedMemoryPaths,
  getAppState,
  setAppState,
  setConversationId,
}: ClearConversationParams): Promise<void>
```

### 清除流程

1. **执行 SessionEnd hooks**：在清除前执行绑定的钩子（超时 1.5s）
2. **缓存驱逐提示**：发送缓存驱逐提示给推理服务
3. **计算保留任务**：识别需要保留的后台任务（`isBackgrounded !== false`）
4. **清空消息**：`setMessages(() => [])`
5. **重置上下文阻塞标志**：如果启用了 PROACTIVE/KAIROS 特性
6. **生成新会话 ID**：使用 `randomUUID()` 创建新 ID
7. **清理会话缓存**：调用 `clearSessionCaches(preservedAgentIds)`
8. **重置工作目录**：`setCwd(getOriginalCwd())`
9. **清理 AppState**：
   - 保留非前台任务
   - 终止前台任务（kill shell、abort controller）
   - 重置归属状态
   - 重置文件历史
   - 重置 MCP 状态
10. **清理计划标识**：`clearAllPlanSlugs()`
11. **清理会话元数据**：`clearSessionMetadata()`
12. **重新生成会话 ID**：`regenerateSessionId({ setCurrentAsParent: true })`
13. **重新指向 Symlink**：为保留的后台任务更新 symlink
14. **保存模式和工作树状态**
15. **执行 SessionStart hooks**：在新会话开始时执行钩子

## 设计要点

1. **任务保留策略**：只清除 `isBackgrounded === false` 的任务
2. **优雅终止**：前台任务被终止时会清理资源（kill、cleanup、clearTimeout）
3. **分析追踪**：记录 `tengu_cache_eviction_hint` 事件
4. **钩子支持**：支持 SessionEnd 和 SessionStart 钩子
5. **Symlink 管理**：确保后台任务的输出链接指向正确的路径

## 与其他文件的关系

- **caches.ts**: 提供 `clearSessionCaches` 函数
- **hooks.ts**: 提供会话钩子执行功能
- **sessionStorage.ts**: 提供会话存储管理
- **AppState.ts**: 应用状态管理
- **index.ts**: 命令注册
