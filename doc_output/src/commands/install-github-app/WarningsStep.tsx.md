# WarningsStep.tsx — 警告展示步骤

> **一句话总结**：显示GitHub CLI检查中的警告信息，允许用户继续或退出。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/WarningsStep.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 73 行 |
| 主要职责 | 显示警告信息和继续选项 |

---

## 功能概述

该组件实现了GitHub App安装流程中的警告展示步骤。当GitHub CLI检查发现问题（如未安装、未认证、缺少权限等）时，显示警告详情，允许用户选择继续或退出修复问题。

---

## 核心内容详解

### 导入与依赖

```typescript
import figures from 'figures'
import * as React from 'react'
import { GITHUB_ACTION_SETUP_DOCS_URL } from '../../constants/github-app.js'
import { Box, Text } from '../../ink.js'
import { useKeybinding } from '../../keybindings/useKeybinding.js'
import type { Warning } from './types.js'
```

### 接口定义

```typescript
interface WarningsStepProps {
  warnings: Warning[]
  onContinue: () => void
}
```

### 主要组件

#### WarningsStep(props): React.ReactNode

- **类型**: React函数组件
- **用途**: 渲染警告信息界面

**显示内容**:

- 标题：警告图标 + "Setup Warnings"
- 说明："We found some potential issues, but you can continue anyway"
- 警告列表：每个警告包含标题、消息和解决步骤
- 操作提示：Enter继续或Ctrl+C退出
- 手动安装文档链接

**警告结构**:

```typescript
type Warning = {
  title: string
  message: string
  instructions: string[]
}
```

---

## 设计要点

1. **非阻塞**: 警告不会阻止用户继续，提供选择余地。

2. **详细信息**: 每个警告都包含标题、详细消息和解决步骤。

3. **清晰指引**: 明确告知用户可以Enter继续或Ctrl+C退出。

4. **手动备选**: 提供手动安装文档链接作为备选方案。

---

## 与其他文件的关系

**依赖**:
- `figures` 库（警告图标）
- `GITHUB_ACTION_SETUP_DOCS_URL` (`../../constants/github-app.js`)
- `Box`, `Text` (`../../ink.js`)
- `useKeybinding` (`../../keybindings/useKeybinding.js`)
- `Warning` 类型 (`./types.js`)

**被依赖**:
- `install-github-app.tsx` - 作为主流程的步骤组件使用

---

## 注意事项

1. **常见警告**:
   - GitHub CLI未安装
   - GitHub CLI未认证
   - 缺少repo或workflow权限

2. **解决步骤**: 提供具体的命令和指引帮助用户修复问题。

3. **继续风险**: 在存在警告的情况下继续可能导致后续步骤失败。
