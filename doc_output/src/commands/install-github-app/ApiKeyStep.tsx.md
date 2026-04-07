# ApiKeyStep.tsx — API密钥配置步骤

> **一句话总结**：实现API密钥选择步骤，支持现有密钥、新密钥和OAuth三种选项。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/ApiKeyStep.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 约80行（编译后） |
| 主要职责 | 提供API密钥选择界面 |

---

## 功能概述

该组件实现了GitHub App安装流程中的API密钥配置步骤。用户可以选择使用现有API密钥、输入新密钥，或通过OAuth流程生成令牌。根据用户已有密钥和OAuth支持情况动态显示可用选项。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import { useCallback, useState } from 'react'
import TextInput from '../../components/TextInput.js'
import { useTerminalSize } from '../../hooks/useTerminalSize.js'
import { Box, color, Text, useTheme } from '../../ink.js'
import { useKeybindings } from '../../keybindings/useKeybinding.js'
```

### 接口定义

```typescript
interface ApiKeyStepProps {
  existingApiKey: string | null
  useExistingKey: boolean
  apiKeyOrOAuthToken: string
  onApiKeyChange: (value: string) => void
  onToggleUseExistingKey: (useExisting: boolean) => void
  onSubmit: () => void
  onCreateOAuthToken?: () => void
  selectedOption?: 'existing' | 'new' | 'oauth'
  onSelectOption?: (option: 'existing' | 'new' | 'oauth') => void
}
```

### 主要组件

#### ApiKeyStep(props): React.ReactNode

- **类型**: React函数组件
- **用途**: 渲染API密钥选择界面

**选项逻辑**:

- **existing**: 显示条件为 `existingApiKey` 存在
- **oauth**: 显示条件为 `onCreateOAuthToken` 回调存在
- **new**: 始终可用

**键盘导航**:

- 左右方向键或特定快捷键在不同选项间切换
- 根据当前选中的选项显示不同的输入界面

---

## 设计要点

1. **动态选项**: 根据用户已有密钥和OAuth支持情况显示不同的选项组合。

2. **选项联动**: 选择OAuth时会自动切换到OAuth流程，选择现有密钥时自动填充。

3. **键盘导航**: 完整的键盘导航支持，无需鼠标操作。

---

## 与其他文件的关系

**依赖**:
- `TextInput` (`../../components/TextInput.js`)
- `useTerminalSize` (`../../hooks/useTerminalSize.js`)
- `useKeybindings` (`../../keybindings/useKeybinding.js`)

**被依赖**:
- `install-github-app.tsx` - 作为主流程的步骤组件使用

---

## 注意事项

1. **密钥安全**: 输入的API密钥不会显示在屏幕上，以安全方式处理。

2. **OAuth优先**: 如果支持OAuth，默认选中OAuth选项（更安全）。
