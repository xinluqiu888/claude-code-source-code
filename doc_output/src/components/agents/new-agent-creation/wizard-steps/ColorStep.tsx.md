# ColorStep.tsx — Agent颜色选择步骤

> **一句话总结**：选择Agent的颜色标识，并准备最终的Agent配置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/wizard-steps/ColorStep.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 选择Agent颜色并准备最终配置 |

---

## 功能概述

`ColorStep`让用户选择Agent的颜色标识，同时整合之前步骤的所有配置，准备最终的Agent定义。

---

## 核心内容详解

### 主要函数

- **ColorStep**: 向导步骤组件
  - **状态**: 使用ColorPicker组件管理颜色选择
  - **处理确认**: 
    - 保存选中的颜色
    - 构建finalAgent对象，包含所有配置信息
    - 进入下一步

### 构建finalAgent

在确认颜色时构建完整的Agent配置：
- `agentType`: Agent类型标识符
- `whenToUse`: 使用场景描述
- `getSystemPrompt`: 系统提示词函数
- `tools`: 选中的工具
- `model`: 选中的模型（可选）
- `color`: 选中的颜色（可选）
- `source`: 存储位置

### UI组件

- **WizardDialogLayout**: 向导对话框布局
- **ColorPicker**: 颜色选择器

---

## 设计要点

- 在此步骤构建完整的finalAgent对象
- 使用ColorPicker的自动颜色作为默认值
- 所有可选字段使用条件展开

---

## 与其他文件的关系

- **依赖**:
  - `useWizard`, `WizardDialogLayout` - 向导框架
  - `ColorPicker` - 颜色选择组件
  - `AgentColorName` - 颜色类型
- **被依赖**:
  - `CreateAgentWizard` - 作为颜色步骤使用
