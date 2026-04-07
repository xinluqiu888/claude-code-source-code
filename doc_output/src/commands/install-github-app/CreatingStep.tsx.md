# CreatingStep.tsx — 创建工作流步骤

> **一句话总结**：显示GitHub Actions工作流创建进度。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/CreatingStep.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 64 行 |
| 主要职责 | 显示工作流创建进度步骤 |

---

## 功能概述

该组件实现了GitHub App安装流程中的工作流创建步骤。显示当前正在执行的操作步骤列表，包括获取仓库信息、创建分支、创建工作流文件、设置密钥、打开PR页面等。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import { Box, Text } from '../../ink.js'
import type { Workflow } from './types.js'
```

### 接口定义

```typescript
interface CreatingStepProps {
  currentWorkflowInstallStep: number
  secretExists: boolean
  useExistingSecret: boolean
  secretName: string
  skipWorkflow?: boolean
  selectedWorkflows: Workflow[]
}
```

### 主要组件

#### CreatingStep(props): React.ReactNode

- **类型**: React函数组件
- **用途**: 渲染创建进度界面

**进度步骤**:

当 `skipWorkflow` 为false时：
1. Getting repository information
2. Creating branch
3. Creating workflow file(s)
4. Setting up secret
5. Opening pull request page

当 `skipWorkflow` 为true时（仅配置密钥）：
1. Getting repository information
2. Setting up secret

**状态显示**:

- **已完成**: 显示绿色对勾
- **进行中**: 显示黄色省略号
- **待处理**: 无特殊标记

---

## 设计要点

1. **动态步骤**: 根据是否跳过工作流创建显示不同的步骤列表。

2. **状态颜色**: 使用不同颜色区分完成、进行中和待处理状态。

3. **复数处理**: 根据选择的工作流数量显示单复数形式（file vs files）。

---

## 与其他文件的关系

**依赖**:
- `Box`, `Text` (`../../ink.js`)
- `Workflow` 类型 (`./types.js`)

**被依赖**:
- `install-github-app.tsx` - 作为主流程的步骤组件使用

---

## 注意事项

1. **步骤索引**: `currentWorkflowInstallStep` 表示当前正在执行的步骤索引。

2. **跳过工作流**: 当用户选择跳过工作流创建时，只显示密钥配置相关步骤。
