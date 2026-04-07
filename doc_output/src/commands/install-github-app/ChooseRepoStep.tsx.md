# ChooseRepoStep.tsx — 仓库选择步骤

> **一句话总结**：允许用户选择当前仓库或输入其他仓库URL。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/ChooseRepoStep.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 约80行（编译后） |
| 主要职责 | 提供仓库选择界面 |

---

## 功能概述

该组件实现了GitHub App安装流程中的仓库选择步骤。用户可以选用当前检测到的仓库，或手动输入其他仓库的URL。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import { useCallback, useState } from 'react'
import TextInput from '../../components/TextInput.js'
import { useTerminalSize } from '../../hooks/useTerminalSize.js'
import { Box, Text } from '../../ink.js'
import { useKeybindings } from '../../keybindings/useKeybinding.js'
```

### 接口定义

```typescript
interface ChooseRepoStepProps {
  currentRepo: string | null
  useCurrentRepo: boolean
  repoUrl: string
  onRepoUrlChange: (value: string) => void
  onToggleUseCurrentRepo: (useCurrentRepo: boolean) => void
  onSubmit: () => void
}
```

### 主要组件

#### ChooseRepoStep(props): React.ReactNode

- **类型**: React函数组件
- **用途**: 渲染仓库选择界面

**功能**:

- 显示当前检测到的仓库（如果有）
- 允许切换使用当前仓库或输入其他仓库
- 输入其他仓库URL时显示文本输入框
- 验证输入不为空

---

## 设计要点

1. **自动检测**: 如果当前目录是Git仓库，自动检测仓库名称。

2. **灵活选择**: 支持使用当前仓库或任意其他仓库。

3. **输入验证**: 提交前验证仓库名称不为空。

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

1. **当前仓库检测**: 依赖 `getGithubRepo` 函数检测当前Git仓库。

2. **URL格式**: 支持 `owner/repo` 格式的仓库名称。
