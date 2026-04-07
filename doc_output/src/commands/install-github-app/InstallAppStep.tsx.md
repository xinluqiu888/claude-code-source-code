# InstallAppStep.tsx — 安装GitHub App步骤

> **一句话总结**：引导用户打开浏览器安装Claude GitHub App。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/InstallAppStep.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 约80行（编译后） |
| 主要职责 | 引导用户安装GitHub App |

---

## 功能概述

该组件实现了GitHub App安装流程中的App安装步骤。显示安装指引，告知用户浏览器将打开Claude GitHub App页面，需要为指定仓库授权。

---

## 核心内容详解

### 导入与依赖

```typescript
import figures from 'figures'
import * as React from 'react'
import { GITHUB_ACTION_SETUP_DOCS_URL } from '../../constants/github-app.js'
import { Box, Text } from '../../ink.js'
import { useKeybinding } from '../../keybindings/useKeybinding.js'
```

### 接口定义

```typescript
interface InstallAppStepProps {
  repoUrl: string
  onSubmit: () => void
}
```

### 主要组件

#### InstallAppStep(props): React.ReactNode

- **类型**: React函数组件
- **用途**: 渲染GitHub App安装引导界面

**显示内容**:

- 标题："Install the Claude GitHub App"
- 浏览器将自动打开的提示
- App安装页面URL（`https://github.com/apps/claude`）
- 需要授权的仓库名称
- 重要提示：确保授权特定仓库
- 按键提示：安装完成后按Enter继续
- 手动安装文档链接

**键盘绑定**:

- Enter: 确认已完成App安装

---

## 设计要点

1. **清晰指引**: 明确告知用户需要执行的操作。

2. **URL显示**: 如果浏览器未自动打开，提供手动访问的URL。

3. **仓库强调**: 突出显示需要授权的仓库名称。

4. **重要提示**: 强调必须授权特定仓库。

---

## 与其他文件的关系

**依赖**:
- `figures` 库（图标）
- `GITHUB_ACTION_SETUP_DOCS_URL` (`../../constants/github-app.js`)
- `Box`, `Text` (`../../ink.js`)
- `useKeybinding` (`../../keybindings/useKeybinding.js`)

**被依赖**:
- `install-github-app.tsx` - 作为主流程的步骤组件使用

---

## 注意事项

1. **浏览器打开**: 浏览器会自动打开App安装页面。

2. **必须授权**: 用户必须显式授权特定仓库，否则工作流无法正常运行。

3. **手动备选**: 如果浏览器未自动打开，提供手动安装的URL。
