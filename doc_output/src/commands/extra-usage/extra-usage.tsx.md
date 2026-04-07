# extra-usage.tsx — 处理额外使用量配置（交互式）

> **一句话总结**：交互式处理额外使用量配置，在浏览器打开后引导用户重新登录。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/extra-usage/extra-usage.tsx` |
| 文件类型 | TypeScript React (TSX) |
| 代码行数 | 16 行 |
| 主要职责 | 处理额外使用量配置的UI流程 |

---

## 功能概述

该文件是 `/extra-usage` 命令的交互式版本入口。当用户执行该命令时，如果浏览器成功打开到使用量管理页面，会显示登录组件让用户重新登录。这是为了确保用户在使用新配置后拥有正确的认证状态。

---

## 核心内容详解

### 导入与依赖

```typescript
import React from 'react'
import type { LocalJSXCommandContext } from '../../commands.js'
import type { LocalJSXCommandOnDone } from '../../types/command.js'
import { Login } from '../login/login.js'
import { runExtraUsage } from './extra-usage-core.js'
```

### 主要函数

#### call(onDone, context): Promise<React.ReactNode | null>

- **类型**: `async function`
- **参数**:
  - `onDone`: `LocalJSXCommandOnDone` - 完成回调
  - `context`: `LocalJSXCommandContext` - 命令上下文，包含 `onChangeAPIKey` 方法
- **返回值**: `Promise<React.ReactNode | null>`
- **用途**: 主入口函数

**执行流程**:

1. 调用 `runExtraUsage()` 执行核心逻辑
2. 如果返回类型为 `'message'`，直接调用 `onDone` 并返回 `null`
3. 如果返回类型为 `'browser-opened'`，渲染 `Login` 组件
4. Login组件完成时：
   - 调用 `context.onChangeAPIKey()` 通知API密钥变更
   - 调用 `onDone` 返回登录成功或中断的消息

---

## 设计要点

1. **与登录流程结合**: 在浏览器打开使用量管理页面后，引导用户重新登录确保认证状态同步。

2. **条件渲染**: 根据 `runExtraUsage` 的结果决定是简单返回消息还是显示登录UI。

3. **API密钥变更通知**: 通过 `onChangeAPIKey` 通知系统认证信息可能已变更。

---

## 与其他文件的关系

**依赖**:
- `Login` 组件 (`../login/login.js`) - 登录UI组件
- `runExtraUsage` (`./extra-usage-core.js`) - 核心逻辑

**被依赖**:
- `extra-usage/index.ts` - 注册为交互式命令

---

## 注意事项

1. **非交互式版本**: 该文件仅用于交互式会话，非交互式版本在 `extra-usage-noninteractive.ts` 中。

2. **Login组件复用**: 直接复用登录命令的Login组件，保持体验一致性。
