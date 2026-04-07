# feedback.tsx — 用户反馈提交界面

> **一句话总结**：提供用户反馈提交界面，支持附带对话上下文。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/feedback/feedback.tsx` |
| 文件类型 | TypeScript React (TSX) |
| 代码行数 | 24 行 |
| 主要职责 | 渲染反馈组件并处理反馈提交流程 |

---

## 功能概述

该文件实现了 `/feedback [report]` 命令（别名 `/bug`），允许用户提交关于Claude Code的反馈。反馈可以附带当前的对话上下文，帮助开发团队理解问题背景。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import type { CommandResultDisplay, LocalJSXCommandContext } from '../../commands.js'
import { Feedback } from '../../components/Feedback.js'
import type { LocalJSXCommandOnDone } from '../../types/command.js'
import type { Message } from '../../types/message.js'
```

### 类型定义

#### BackgroundTasks

- **类型**: `Record<string, object>`
- **结构**:
  ```typescript
  {
    [taskId: string]: {
      type: string
      identity?: { agentId: string }
      messages?: Message[]
    }
  }
  ```

### 主要函数

#### renderFeedbackComponent(onDone, abortSignal, messages, initialDescription?, backgroundTasks?): React.ReactNode

- **类型**: `function`
- **参数**:
  - `onDone`: 完成回调（可选结果和显示选项）
  - `abortSignal`: `AbortSignal` - 用于取消操作
  - `messages`: `Message[]` - 当前对话消息
  - `initialDescription`: `string` (可选) - 初始描述文本
  - `backgroundTasks`: `BackgroundTasks` (可选) - 后台任务信息
- **返回值**: `React.ReactNode`
- **用途**: 渲染Feedback组件的共享函数

#### call(onDone, context, args?): Promise<React.ReactNode>

- **类型**: `async function`
- **参数**:
  - `onDone`: 完成回调
  - `context`: 命令上下文
  - `args`: 可选的初始描述参数
- **返回值**: `Promise<React.ReactNode>`
- **用途**: 主入口函数
- **逻辑**: 提取args作为初始描述，调用 `renderFeedbackComponent`

---

## 设计要点

1. **可复用组件**: `renderFeedbackComponent` 作为独立函数导出，便于在其他地方复用。

2. **上下文携带**: 反馈可以包含当前对话消息和后台任务信息，帮助开发团队定位问题。

3. **取消支持**: 通过 `AbortSignal` 支持取消正在进行的反馈提交。

4. **可选描述**: 用户可以直接在命令后附带描述（如 `/bug 遇到了崩溃问题`），减少输入步骤。

---

## 与其他文件的关系

**依赖**:
- `Feedback` 组件 (`../../components/Feedback.js`) - 反馈表单UI
- `CommandResultDisplay` 类型 (`../../commands.js`)
- `Message` 类型 (`../../types/message.js`)

**被依赖**:
- `feedback/index.ts` - 导出为命令配置

---

## 注意事项

1. **共享函数**: `renderFeedbackComponent` 的设计表明反馈功能可能需要在其他地方调用，如错误处理流程。

2. **后台任务**: `backgroundTasks` 参数支持包含后台任务信息，说明系统支持某种形式的后台任务/代理。

3. **AbortController**: 使用AbortSignal支持取消操作，说明反馈提交流程可能涉及异步网络请求。
