# useCancelRequest.ts — 取消请求处理组件

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/useCancelRequest.ts`
- **类型**: React 组件（Hook 形式）
- **导出组件**: `CancelRequestHandler`
- **依赖**: keybindings, AppState, notifications

## 功能概述

本组件处理所有取消/中断相关的键盘快捷键：
1. Escape (chat:cancel): 取消当前任务或弹出队列命令
2. Ctrl+C (app:interrupt): 中断并终止所有后台 Agent
3. Ctrl+X Ctrl+K (chat:killAgents): 双次按键终止所有后台 Agent

## 核心内容详解

### 优先级处理

**handleCancel 优先级**
```
1. 有运行中任务 → 取消任务
2. 队列中有命令 → 弹出队列
3. 其他 → 通用取消
```

### isActive 条件

**Escape (isEscapeActive)**
```typescript
isContextActive &&
(canCancelRunningTask || hasQueuedCommands) &&
!isInSpecialModeWithEmptyInput &&
!isViewingTeammate
```

**Ctrl+C (isCtrlCActive)**
```typescript
isContextActive &&
(canCancelRunningTask || hasQueuedCommands || isViewingTeammate)
```

### 终止 Agent 双次确认

**KILL_AGENTS_CONFIRM_WINDOW_MS = 3000**

第一次按键：显示确认提示
```typescript
addNotification({
  key: 'kill-agents-confirm',
  text: `Press ${shortcut} again to stop background agents`,
  timeoutMs: KILL_AGENTS_CONFIRM_WINDOW_MS,
})
```

第二次按键（3秒内）：执行终止

### killAllAgentsAndNotify

终止所有运行中的 local_agent 任务：
1. 调用 `killAllRunningAgentTasks`
2. 标记所有 Agent 已通知
3. 发送 SDK 终止事件
4. 入队聚合通知消息

## 设计要点

### 1. 上下文守卫

避免在其他上下文中拦截 Escape：
- 历史搜索
- 帮助界面
- 覆盖层（ModelPicker 等）
- Vim 插入模式

### 2. 特殊模式处理

空输入时的特殊模式（bash/background）：
- Escape：退出模式（由 PromptInput 处理）
- Ctrl+C：仍然取消任务

### 3. 快捷键区分

| 快捷键 | 行为 |
|--------|------|
| Escape | 温和取消，优先任务后队列 |
| Ctrl+C | 强制中断，包含终止 Agent |
| Ctrl+X Ctrl+K | 专门用于终止 Agent，需确认 |

## 与其他文件的关系

- **useKeybinding**: 快捷键注册
- **LocalAgentTask.ts**: Agent 终止逻辑
- **messageQueueManager.ts**: 命令队列管理
- **notifications.ts**: 通知系统

## 注意事项

1. **必须渲染在 KeybindingSetup 内**: 需要快捷键上下文
2. **渲染 null**: 纯逻辑组件，无 UI
3. **Ctrl+X 始终激活**: 作为和弦前缀，不能设为 inactive
4. **Vim 模式特殊处理**: 插入模式下 Escape 切换到正常模式，不取消
