# exit.tsx — 处理退出REPL流程

> **一句话总结**：处理用户退出Claude Code REPL的流程，支持普通退出、后台会话分离和工作区退出确认。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/exit/exit.tsx` |
| 文件类型 | TypeScript React (TSX) |
| 代码行数 | 32 行 |
| 主要职责 | 实现退出命令的核心逻辑，处理不同退出场景 |

---

## 功能概述

该文件实现了 `/exit` 和 `/quit` 命令的核心功能。当用户想要退出Claude Code时，此模块负责处理各种退出场景：

1. **后台会话退出**：当用户在 `claude --bg` 后台tmux会话中时，执行tmux detach操作而非直接终止进程，允许用户稍后通过 `claude attach` 重新连接。

2. **工作区会话退出**：如果用户当前处于git worktree会话中，显示ExitFlow组件让用户确认是否保留工作区。

3. **普通退出**：显示随机告别消息，并执行优雅关机流程。

---

## 核心内容详解

### 导入与依赖

```typescript
import { feature } from 'bun:bundle'          // Bun运行时特性标志
import { spawnSync } from 'child_process'     // 用于执行tmux detach
import sample from 'lodash-es/sample.js'      // 随机选择告别消息
import * as React from 'react'                // React框架
import { ExitFlow } from '../../components/ExitFlow.js'
import type { LocalJSXCommandOnDone } from '../../types/command.js'
import { isBgSession } from '../../utils/concurrentSessions.js'
import { gracefulShutdown } from '../../utils/gracefulShutdown.js'
import { getCurrentWorktreeSession } from '../../utils/worktree.js'
```

### 常量定义

- **GOODBYE_MESSAGES**: `string[]` - 告别消息数组，包含 "Goodbye!", "See ya!", "Bye!", "Catch you later!"

### 主要函数

#### getRandomGoodbyeMessage()

- **类型**: `function`
- **返回值**: `string`
- **用途**: 从GOODBYE_MESSAGES数组中随机选择一条告别消息
- **关键逻辑**: 使用 lodash 的 sample 函数，如果失败则返回默认值 'Goodbye!'

#### call(onDone: LocalJSXCommandOnDone): Promise<React.ReactNode>

- **类型**: `async function`
- **参数**:
  - `onDone`: `LocalJSXCommandOnDone` - 命令完成时的回调函数
- **返回值**: `Promise<React.ReactNode>`
- **用途**: 主入口函数，处理退出逻辑

**执行流程**:

1. **检查后台会话**: 如果 `BG_SESSIONS` 特性启用且当前是后台会话 (`isBgSession()`)
   - 调用 `onDone()` 完成命令
   - 执行 `tmux detach-client` 分离客户端
   - 返回 `null` (不渲染UI)

2. **检查工作区会话**: 如果当前在worktree会话中 (`getCurrentWorktreeSession() !== null`)
   - 返回 `<ExitFlow />` 组件，让用户确认是否保留工作区

3. **普通退出**:
   - 显示随机告别消息
   - 调用 `gracefulShutdown(0, 'prompt_input_exit')` 优雅关机
   - 返回 `null`

---

## 设计要点

1. **多场景处理**: 通过条件判断优雅处理三种不同退出场景，代码结构清晰。

2. **用户体验**: 使用随机告别消息增加亲切感；后台会话分离而非终止，保持会话可恢复。

3. **组件化**: 复杂的退出确认逻辑委托给 `ExitFlow` 组件，保持主函数简洁。

4. **错误安全**: 使用 `??` 运算符确保始终有默认告别消息。

---

## 与其他文件的关系

**依赖**:
- `ExitFlow` 组件 (`../../components/ExitFlow.js`) - 工作区退出确认UI
- `isBgSession` (`../../utils/concurrentSessions.js`) - 检测后台会话
- `gracefulShutdown` (`../../utils/gracefulShutdown.js`) - 优雅关机
- `getCurrentWorktreeSession` (`../../utils/worktree.js`) - 获取工作区会话

**被依赖**:
- `exit/index.ts` - 导出为命令配置

---

## 注意事项

1. **BG_SESSIONS特性**: 后台会话功能由 `feature('BG_SESSIONS')` 控制，需要Bun运行时支持。

2. **tmux依赖**: 后台会话功能依赖系统中已安装tmux。

3. **ExitFlow回调**: ExitFlow组件接收 `onDone` 和 `onCancel` 两个回调，都在不同场景下调用主回调。

4. **异步处理**: 虽然 `gracefulShutdown` 是异步的，但函数返回null后，React会处理组件卸载。
