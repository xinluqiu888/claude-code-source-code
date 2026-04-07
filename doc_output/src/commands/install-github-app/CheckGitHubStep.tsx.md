# CheckGitHubStep.tsx — GitHub CLI检查步骤

> **一句话总结**：显示GitHub CLI检查状态。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/CheckGitHubStep.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 14 行 |
| 主要职责 | 显示GitHub CLI检查状态提示 |

---

## 功能概述

该组件实现了GitHub App安装流程中的GitHub CLI检查步骤。显示简单的文本提示，告知用户正在检查GitHub CLI安装状态。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import { Text } from '../../ink.js'
```

### 主要组件

#### CheckGitHubStep(): React.ReactNode

- **类型**: React函数组件（无参数）
- **用途**: 显示检查状态文本

**显示内容**: "Checking GitHub CLI installation…"

---

## 与其他文件的关系

**依赖**:
- `Text` (`../../ink.js`)

**被依赖**:
- `install-github-app.tsx` - 作为主流程的步骤组件使用

---

## 注意事项

1. **简单组件**: 这是一个纯展示组件，仅显示静态文本。

2. **过渡步骤**: 此步骤很快会被后续的实际检查结果替代。
