# PromptStep.tsx — Agent系统提示词步骤

> **一句话总结**：输入Agent的系统提示词（systemPrompt）。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/wizard-steps/PromptStep.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 输入Agent的系统提示词 |

---

## 功能概述

`PromptStep`接收用户输入的Agent系统提示词（systemPrompt），这是定义Agent行为和角色的核心指令。

---

## 核心内容详解

### 主要函数

- **PromptStep**: 向导步骤组件
  - **状态**:
    - `systemPrompt`: 系统提示词文本
    - `cursorOffset`: 光标位置
    - `error`: 验证错误
  - **验证**: 非空检查
  - **提交**: 保存到wizardData并进入下一步

### 外部编辑器支持

- 支持Ctrl+G打开外部编辑器
- 适合输入长文本

### UI组件

- **WizardDialogLayout**: 向导对话框布局
- **TextInput**: 多行文本输入

---

## 设计要点

- 提示"Be comprehensive for best results"
- 支持长文本输入
- 外部编辑器支持

---

## 与其他文件的关系

- **依赖**:
  - `useWizard`, `WizardDialogLayout` - 向导框架
  - `TextInput` - 文本输入组件
  - `editPromptInEditor` - 外部编辑器
- **被依赖**:
  - `CreateAgentWizard` - 作为提示词步骤使用
