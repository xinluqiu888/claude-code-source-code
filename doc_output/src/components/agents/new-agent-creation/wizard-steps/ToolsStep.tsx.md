# ToolsStep.tsx — Agent工具选择步骤

> **一句话总结**：选择Agent可用的工具集。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/wizard-steps/ToolsStep.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 选择Agent可用的工具 |

---

## 功能概述

`ToolsStep`使用`ToolSelector`组件让用户选择Agent可用的工具集。

---

## 核心内容详解

### Props

- **tools**: 可用工具集

### 主要函数

- **ToolsStep**: 向导步骤组件
  - **处理完成**: 保存选中的工具到wizardData
  - **传递**: 将tools和initialTools传递给ToolSelector

### 数据流

1. 从wizardData获取已选择的工具（initialTools）
2. 渲染ToolSelector组件
3. 用户选择完成后更新wizardData
4. 进入下一步

### 特殊处理

- 支持undefined表示"所有工具"
- ToolSelector内部会展开*通配符

---

## 设计要点

- 复用现有的ToolSelector组件
- 支持返回上一步
- 保持"所有工具"的语义

---

## 与其他文件的关系

- **依赖**:
  - `useWizard`, `WizardDialogLayout` - 向导框架
  - `ToolSelector` - 工具选择组件
- **被依赖**:
  - `CreateAgentWizard` - 作为工具步骤使用
