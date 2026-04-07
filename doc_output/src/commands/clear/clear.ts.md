# clear.ts — clear 命令本地实现

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/clear/clear.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 7 行 |
| 主要职责 | clear 命令的轻量级入口，委托给 conversation.ts 实现 |

## 功能概述

该文件是 `/clear` 命令的本地（非 JSX）实现入口。它非常简单，只是将调用委托给 `clearConversation` 函数。这种分离允许主要的清除逻辑在需要时懒加载。

## 核心内容详解

### 实现

```typescript
import type { LocalCommandCall } from '../../types/command.js'
import { clearConversation } from './conversation.js'

export const call: LocalCommandCall = async (_, context) => {
  await clearConversation(context)
  return { type: 'text', value: '' }
}
```

### 返回值

- 类型：`'text'`
- 值：空字符串（清除命令不需要输出文本）

## 设计要点

1. **委托模式**：将实际逻辑委托给 `conversation.ts` 中的 `clearConversation`
2. **轻量级**：文件本身非常小，便于快速加载
3. **统一接口**：符合 `LocalCommandCall` 接口规范

## 与其他文件的关系

- **conversation.ts**: 提供 `clearConversation` 函数，包含实际的清除逻辑
- **index.ts**: 命令注册配置
