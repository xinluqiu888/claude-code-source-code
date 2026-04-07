# fast.tsx — 快速模式切换与管理

> **一句话总结**：实现快速模式(Fast Mode)的开关控制和交互式选择界面。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/fast/fast.tsx` |
| 文件类型 | TypeScript React (TSX) |
| 代码行数 | 268 行（React Compiler优化后代码） |
| 主要职责 | 提供快速模式的开启/关闭控制界面和状态管理 |

---

## 功能概述

该文件实现了 `/fast [on|off]` 命令，用于控制Claude Code的快速模式。快速模式使用特定的高性能模型（如Opus 4.6），以额外使用量计费，提供更快的响应速度。

主要功能：
- 显示快速模式的当前状态和配置选项
- 支持通过参数直接开关 (`/fast on` 或 `/fast off`)
- 交互式界面允许用户确认切换
- 自动处理模型切换（如果当前模型不支持快速模式）
- 显示定价信息和冷却状态

---

## 核心内容详解

### 导入与依赖

```typescript
import { c as _c } from "react/compiler-runtime"          // React Compiler运行时
import { feature } from 'bun:bundle'                       // Bun特性标志
import * as React from 'react'
import { useState } from 'react'
import type { CommandResultDisplay, LocalJSXCommandContext } from '../../commands.js'
import { Dialog } from '../../components/design-system/Dialog.js'
import { FastIcon, getFastIconString } from '../../components/FastIcon.js'
import { Box, Link, Text } from '../../ink.js'
import { useKeybindings } from '../../keybindings/useKeybinding.js'
import { type AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS, logEvent } from '../../services/analytics/index.js'
import { type AppState, useAppState, useSetAppState } from '../../state/AppState.js'
import type { LocalJSXCommandOnDone } from '../../types/command.js'
import { clearFastModeCooldown, FAST_MODE_MODEL_DISPLAY, getFastModeModel, getFastModeRuntimeState, getFastModeUnavailableReason, isFastModeEnabled, isFastModeSupportedByModel, prefetchFastModeStatus } from '../../utils/fastMode.js'
import { formatDuration } from '../../utils/format.js'
import { formatModelPricing, getOpus46CostTier } from '../../utils/modelCost.js'
import { updateSettingsForSource } from '../../utils/settings/settings.js'
```

### 主要函数

#### applyFastMode(enable, setAppState): void

- **类型**: `function`
- **参数**:
  - `enable`: `boolean` - 是否启用快速模式
  - `setAppState`: 状态更新函数
- **用途**: 应用快速模式设置到应用状态
- **关键逻辑**:
  - 清除快速模式冷却状态
  - 更新用户设置
  - 如果启用且当前模型不支持，自动切换到支持快速模式的模型

#### FastModePicker(props): React.ReactNode

- **类型**: `React Component`
- **参数**:
  - `onDone`: 完成回调
  - `unavailableReason`: 快速模式不可用的原因（如有）
- **用途**: 渲染快速模式选择器对话框
- **功能**:
  - 显示当前快速模式状态（开/关）
  - 显示定价信息
  - 显示冷却状态（如被限制）
  - 提供确认/取消操作
  - 键盘快捷键支持（Tab切换，Enter确认，Esc取消）

#### handleFastModeShortcut(enable, getAppState, setAppState): Promise<string>

- **类型**: `async function`
- **参数**:
  - `enable`: `boolean` - 目标状态
  - `getAppState`: 获取应用状态
  - `setAppState`: 设置应用状态
- **返回值**: `Promise<string>` - 结果消息
- **用途**: 处理快捷方式切换（`/fast on` 或 `/fast off`）

#### call(onDone, context, args?): Promise<React.ReactNode | null>

- **类型**: `async function`
- **参数**:
  - `onDone`: 完成回调
  - `context`: 命令上下文
  - `args`: 可选参数（'on' 或 'off'）
- **返回值**: `Promise<React.ReactNode | null>`
- **用途**: 主入口函数

**执行流程**:
1. 检查快速模式是否启用，如未启用返回 `null`
2. 预获取快速模式状态
3. 解析参数：
   - `on` 或 `off`: 直接切换并返回结果消息
   - 无参数: 显示 `FastModePicker` 交互界面

---

## 设计要点

1. **模型自动切换**: 当用户启用快速模式但当前模型不支持时，自动切换到支持的模型。

2. **冷却状态显示**: 如果快速模式因限制而冷却，显示剩余时间。

3. **定价透明**: 在UI中明确显示快速模式的计费方式。

4. **键盘友好**: 支持Tab切换、Enter确认、Esc取消的键盘操作。

5. **React Compiler优化**: 代码经过React Compiler优化，包含大量memo缓存逻辑。

---

## 与其他文件的关系

**依赖**:
- `Dialog` 组件 (`../../components/design-system/Dialog.js`)
- `FastIcon` 组件 (`../../components/FastIcon.js`)
- `useAppState`, `useSetAppState` (`../../state/AppState.js`)
- 快速模式工具函数 (`../../utils/fastMode.js`)
- 定价工具 (`../../utils/modelCost.js`)

**被依赖**:
- `fast/index.ts` - 导出为命令配置

---

## 注意事项

1. **React Compiler**: 代码使用React Compiler优化，包含大量 `_c` 调用和缓存检查。

2. **特性标志**: 快速模式功能可能由 `feature('...')` 控制。

3. **参数简化**: 支持简单的 `on`/`off` 参数直接切换，无需交互。

4. **定价显示**: 使用 `formatModelPricing` 和 `getOpus46CostTier` 显示当前定价。

5. **冷却处理**: `clearFastModeCooldown` 在切换时重置冷却状态。
