# MethodStep.tsx — Agent创建方式选择步骤

> **一句话总结**：选择Agent创建方式：AI自动生成或手动配置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/wizard-steps/MethodStep.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 选择Agent创建方式 |

---

## 功能概述

`MethodStep`是Agent创建向导的第二步，让用户选择创建方式：
- Generate with Claude (recommended): AI自动生成
- Manual configuration: 手动逐步配置

根据选择动态导航到不同的步骤。

---

## 核心内容详解

### 主要函数

- **MethodStep**: 向导步骤组件
  - **选项**:
    - `generate`: Generate with Claude (recommended)
    - `manual`: Manual configuration
  - **动态导航**:
    - 选择generate -> 进入GenerateStep (index 2)
    - 选择manual -> 跳过AI生成，直接到TypeStep (index 3)

### 状态更新

- 更新`method`字段
- 设置`wasGenerated`标志（true/false）

### 使用向导API

- `goNext`: 进入下一步
- `goToStep(3)`: 跳转到指定步骤
- `updateWizardData`: 更新创建方式和生成标志

---

## 设计要点

- 动态导航逻辑，根据选择跳转不同步骤
- 设置`wasGenerated`标志供后续步骤使用
- 支持返回上一步

---

## 与其他文件的关系

- **依赖**:
  - `useWizard`, `WizardDialogLayout` - 向导框架
  - `Select` - 选择组件
- **被依赖**:
  - `CreateAgentWizard` - 作为第2步使用
