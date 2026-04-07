# mobile.tsx — 移动应用QR码展示

> **一句话总结**：生成并展示Claude移动应用下载二维码，支持iOS/Android平台切换。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/mobile/mobile.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 274 行 |
| 主要职责 | 生成QR码并提供平台切换界面 |

---

## 功能概述

该文件实现了 `/mobile` 命令的功能。生成iOS和Android应用商店的QR码，提供交互式界面让用户在两个平台之间切换，支持键盘快捷键操作。

---

## 核心内容详解

### 导入与依赖

```typescript
import { toString as qrToString } from 'qrcode'
import * as React from 'react'
import { useCallback, useEffect, useState } from 'react'
import { Pane } from '../../components/design-system/Pane.js'
import type { KeyboardEvent } from '../../ink/events/keyboard-event.js'
import { Box, Text } from '../../ink.js'
import { useKeybinding } from '../../keybindings/useKeybinding.js'
import type { LocalJSXCommandOnDone } from '../../types/command.js'
```

### 常量定义

```typescript
type Platform = 'ios' | 'android'

const PLATFORMS: Record<Platform, { url: string }> = {
  ios: {
    url: 'https://apps.apple.com/app/claude-by-anthropic/id6473753684'
  },
  android: {
    url: 'https://play.google.com/store/apps/details?id=com.anthropic.claude'
  }
}
```

### 主要组件

#### MobileQRCode(props): React.ReactNode

- **类型**: React函数组件
- **参数**:
  - `onDone`: `() => void` - 关闭回调
- **用途**: QR码展示和平台切换组件

**状态管理**:

- `platform`: 当前选中的平台（'ios' 或 'android'）
- `qrCodes`: 存储预生成的iOS和Android QR码

**键盘处理**:

- `q` 或 `Ctrl+C`: 退出
- `Tab` 或 `←/→`: 切换平台

**useEffect**:

- 组件挂载时并行生成两个平台的QR码，避免切换时的闪烁

#### call(onDone): Promise<React.ReactNode>

- **类型**: `async function`
- **返回值**: `Promise<React.ReactNode>`
- **用途**: 命令入口函数

---

## 设计要点

1. **预生成QR码**: 在组件挂载时同时生成两个平台的QR码，避免切换时的延迟和闪烁。

2. **UTF8输出**: 使用 `type: 'utf8'` 生成终端可显示的QR码。

3. **键盘快捷键**: 支持 `q`、Tab、方向键等多种方式操作。

4. **React Compiler优化**: 使用React Compiler的缓存机制优化渲染性能。

---

## 与其他文件的关系

**依赖**:
- `qrcode` 库（QR码生成）
- `Pane` (`../../components/design-system/Pane.js`)
- `useKeybinding` (`../../keybindings/useKeybinding.js`)

**被依赖**:
- `mobile/index.ts` - 导出为命令配置

---

## 注意事项

1. **QR码库**: 依赖 `qrcode` npm包生成QR码。

2. **错误处理**: QR码生成失败时静默处理，显示空内容。

3. **平台链接**: iOS使用App Store链接，Android使用Google Play链接。
