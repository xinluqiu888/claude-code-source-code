# CheckExistingSecretStep.tsx — 现有密钥检查步骤

> **一句话总结**：检查仓库是否已有ANTHROPIC_API_KEY密钥，提供使用或替换选项。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/CheckExistingSecretStep.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 约80行（编译后） |
| 主要职责 | 处理已存在密钥的情况 |

---

## 功能概述

该组件实现了GitHub App安装流程中的现有密钥检查步骤。如果仓库已存在 `ANTHROPIC_API_KEY` 密钥，询问用户是使用现有密钥还是配置新的密钥名称。

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
interface CheckExistingSecretStepProps {
  useExistingSecret: boolean
  secretName: string
  onToggleUseExistingSecret: (useExisting: boolean) => void
  onSecretNameChange: (value: string) => void
  onSubmit: () => void
}
```

### 主要组件

#### CheckExistingSecretStep(props): React.ReactNode

- **类型**: React函数组件
- **用途**: 渲染现有密钥选择界面

**选项**:

- **使用现有密钥**: 直接使用已存在的 `ANTHROPIC_API_KEY`
- **使用新密钥名称**: 输入自定义密钥名称（如 `CLAUDE_API_KEY`）

---

## 设计要点

1. **密钥复用**: 允许复用已存在的密钥，避免重复配置。

2. **自定义名称**: 支持自定义密钥名称，适应不同的命名规范。

3. **键盘操作**: 完整的键盘导航支持。

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

1. **默认密钥名**: 默认密钥名称为 `ANTHROPIC_API_KEY`。

2. **兼容性**: 自定义密钥名称需要确保与Workflow文件中的引用一致。
