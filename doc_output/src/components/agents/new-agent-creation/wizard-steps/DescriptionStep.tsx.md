# DescriptionStep.tsx — Agent描述输入步骤

> **一句话总结**：输入Agent的使用场景描述（whenToUse）。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/wizard-steps/DescriptionStep.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 输入Agent的使用场景描述 |

---

## 功能概述

`DescriptionStep`接收用户输入的Agent使用场景描述（whenToUse），这是告诉Claude何时使用该Agent的描述性文本。

---

## 核心内容详解

### 主要函数

- **DescriptionStep**: 向导步骤组件
  - **状态**:
    - `whenToUse`: 描述文本
    - `cursorOffset`: 光标位置
    - `error`: 验证错误
  - **验证**: 非空检查
  - **提交**: 保存到wizardData并进入下一步

### 外部编辑器支持

- 支持Ctrl+G打开外部编辑器
- 使用`editPromptInEditor`函数
- 编辑完成后更新文本和光标位置

### UI组件

- **WizardDialogLayout**: 向导对话框布局
- **TextInput**: 多行文本输入
- **Byline**: 快捷键提示

---

## 设计要点

- 支持长文本输入（80列宽度）
- 外部编辑器支持
- 非空验证

---

## 与其他文件的关系

- **依赖**:
  - `useWizard`, `WizardDialogLayout` - 向导框架
  - `TextInput` - 文本输入组件
  - `editPromptInEditor` - 外部编辑器
- **被依赖**:
  - `CreateAgentWizard` - 作为描述步骤使用
