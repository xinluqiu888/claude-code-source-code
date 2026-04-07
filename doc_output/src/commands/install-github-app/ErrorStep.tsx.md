# ErrorStep.tsx — 错误展示步骤

> **一句话总结**：显示安装过程中的错误信息和解决建议。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/ErrorStep.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 80 行（编译后） |
| 主要职责 | 显示错误信息和帮助链接 |

---

## 功能概述

该组件实现了GitHub App安装流程中的错误展示步骤。当安装过程中发生错误时，显示详细的错误信息、错误原因、解决建议，以及指向手动安装文档的链接。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import { GITHUB_ACTION_SETUP_DOCS_URL } from '../../constants/github-app.js'
import { Box, Text } from '../../ink.js'
```

### 接口定义

```typescript
interface ErrorStepProps {
  error: string | undefined
  errorReason?: string
  errorInstructions?: string[]
}
```

### 主要组件

#### ErrorStep(props): React.ReactNode

- **类型**: React函数组件
- **用途**: 渲染错误信息界面

**显示内容**:

- 标题："Install GitHub App"
- 错误信息：红色高亮显示
- 错误原因（如果有）
- 解决步骤（如果有）
- 手动安装文档链接

---

## 设计要点

1. **清晰分层**: 错误信息、原因、解决步骤分层显示。

2. **帮助链接**: 始终提供指向手动安装文档的链接。

3. **键盘退出**: 提示用户可以按任意键退出。

---

## 与其他文件的关系

**依赖**:
- `GITHUB_ACTION_SETUP_DOCS_URL` (`../../constants/github-app.js`)
- `Box`, `Text` (`../../ink.js`)

**被依赖**:
- `install-github-app.tsx` - 作为主流程的步骤组件使用

---

## 注意事项

1. **错误原因**: 某些错误会提供具体的原因说明。

2. **解决步骤**: 提供可操作的解决步骤列表。

3. **文档链接**: 指向 `https://github.com/anthropics/claude-code-action` 的手动安装说明。
