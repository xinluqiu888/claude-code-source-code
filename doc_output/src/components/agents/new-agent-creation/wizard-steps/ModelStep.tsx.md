# ModelStep.tsx — Agent模型选择步骤

> **一句话总结**：选择Agent使用的AI模型。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/wizard-steps/ModelStep.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 选择Agent使用的AI模型 |

---

## 功能概述

`ModelStep`使用`ModelSelector`组件让用户选择Agent使用的AI模型。

---

## 核心内容详解

### 主要函数

- **ModelStep**: 向导步骤组件
  - **处理完成**: 保存选中的模型到wizardData
  - **传递**: 将initialModel传递给ModelSelector

### 数据流

1. 从wizardData获取已选择的模型（selectedModel）
2. 渲染ModelSelector组件
3. 用户选择完成后更新wizardData
4. 进入下一步

---

## 设计要点

- 复用现有的ModelSelector组件
- 支持返回上一步
- 可选步骤（可以不选择）

---

## 与其他文件的关系

- **依赖**:
  - `useWizard`, `WizardDialogLayout` - 向导框架
  - `ModelSelector` - 模型选择组件
- **被依赖**:
  - `CreateAgentWizard` - 作为模型步骤使用
