# ExistingWorkflowStep.tsx — 现有工作流处理步骤

> **一句话总结**：当检测到现有Claude工作流时，提供更新、跳过或退出选项。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/ExistingWorkflowStep.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 约80行（编译后） |
| 主要职责 | 处理已存在工作流的情况 |

---

## 功能概述

该组件实现了GitHub App安装流程中的现有工作流处理步骤。当检测到仓库已存在Claude工作流文件时，询问用户是更新到最新版本、仅配置密钥，还是退出不执行任何操作。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import { Select } from 'src/components/CustomSelect/index.js'
import { Box, Text } from '../../ink.js'
```

### 接口定义

```typescript
interface ExistingWorkflowStepProps {
  repoName: string
  onSelectAction: (action: 'update' | 'skip' | 'exit') => void
}
```

### 主要组件

#### ExistingWorkflowStep(props): React.ReactNode

- **类型**: React函数组件
- **用途**: 渲染现有工作流处理界面

**选项**:

1. **Update workflow file with latest version**: 更新工作流到最新版本
2. **Skip workflow update (configure secrets only)**: 跳过工作流更新，仅配置密钥
3. **Exit without making changes**: 退出，不做任何更改

---

## 设计要点

1. **智能检测**: 自动检测 `.github/workflows/claude.yml` 是否存在。

2. **灵活处理**: 提供多种处理方式，满足不同场景需求。

3. **选择组件**: 使用 `Select` 组件提供友好的选项界面。

---

## 与其他文件的关系

**依赖**:
- `Select` (`src/components/CustomSelect/index.js`)
- `Box`, `Text` (`../../ink.js`)

**被依赖**:
- `install-github-app.tsx` - 作为主流程的步骤组件使用

---

## 注意事项

1. **仅检测Claude工作流**: 只检测特定路径的Claude工作流文件。

2. **密钥配置**: 选择"仅配置密钥"时，仍然可以配置API密钥。
