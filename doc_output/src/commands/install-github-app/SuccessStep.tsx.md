# SuccessStep.tsx — 安装成功步骤

> **一句话总结**：显示GitHub Actions安装成功的信息和后续步骤。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-github-app/SuccessStep.tsx` |
| 文件类型 | TypeScript TSX |
| 代码行数 | 约80行（编译后） |
| 主要职责 | 显示安装成功信息和后续指引 |

---

## 功能概述

该组件实现了GitHub App安装流程中的成功步骤。显示工作流创建成功、密钥配置完成的确认信息，以及用户接下来需要执行的步骤。

---

## 核心内容详解

### 导入与依赖

```typescript
import * as React from 'react'
import { Box, Text } from '../../ink.js'
```

### 接口定义

```typescript
type SuccessStepProps = {
  secretExists: boolean
  useExistingSecret: boolean
  secretName: string
  skipWorkflow?: boolean
}
```

### 主要组件

#### SuccessStep(props): React.ReactNode

- **类型**: React函数组件
- **用途**: 渲染成功信息界面

**显示内容**:

- 标题："Install GitHub App" + "Success"
- 工作流创建成功状态（如果未跳过）
- 密钥配置状态（使用现有或新建）
- 后续步骤列表

**后续步骤**（完整安装）：
1. A pre-filled PR page has been created
2. Install the Claude GitHub App if you haven't already
3. Merge the PR to enable Claude PR assistance

**后续步骤**（仅配置密钥）：
1. Install the Claude GitHub App if you haven't already
2. Your workflow file was kept unchanged
3. API key is configured and ready to use

---

## 设计要点

1. **清晰总结**: 清楚列出已完成和待完成的步骤。

2. **条件显示**: 根据是否跳过工作流显示不同的后续步骤。

3. **成功标识**: 使用绿色对勾标识成功的操作。

---

## 与其他文件的关系

**依赖**:
- `Box`, `Text` (`../../ink.js`)

**被依赖**:
- `install-github-app.tsx` - 作为主流程的步骤组件使用

---

## 注意事项

1. **PR创建**: 成功后会创建一个预填充的PR页面。

2. **App安装**: 提醒用户如果尚未安装GitHub App需要先安装。

3. **合并PR**: 必须合并PR才能使工作流生效。
