# TypeStep.tsx — Agent类型输入步骤

> **一句话总结**：输入Agent的唯一标识符（类型名称）。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/wizard-steps/TypeStep.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 输入并验证Agent类型标识符 |

---

## 功能概述

`TypeStep`是Agent创建向导的步骤，让用户输入Agent的唯一标识符：
- 接收用户输入
- 实时验证标识符格式
- 显示验证错误信息

---

## 核心内容详解

### Props

- **existingAgents**: 已存在的Agent列表（用于验证重复）

### 主要函数

- **TypeStep**: 向导步骤组件
  - **状态**:
    - `agentType`: 输入的Agent类型
    - `error`: 验证错误信息
    - `cursorOffset`: 光标位置
  - **验证**: 调用`validateAgentType`进行格式验证
  - **提交**: 验证通过后更新wizardData并进入下一步

### 验证规则

通过`validateAgentType`函数验证：
- 不能为空
- 必须以字母数字开头和结尾
- 只能包含字母、数字和连字符
- 长度3-50字符

### 使用向导API

- `useWizard<AgentWizardData>()`: 获取向导上下文
- `wizardData.agentType`: 获取已保存的类型
- `useKeybinding`: 绑定Esc键返回

---

## 设计要点

- 实时显示验证错误
- 支持键盘导航（Esc返回）
- 从wizardData恢复已输入的值
- 使用TextInput组件提供编辑功能

---

## 与其他文件的关系

- **依赖**:
  - `useWizard`, `WizardDialogLayout` - 向导框架
  - `TextInput` - 文本输入组件
  - `validateAgentType` - 验证函数
  - `useKeybinding` - 键盘绑定
- **被依赖**:
  - `CreateAgentWizard` - 作为第4步使用（手动模式）
