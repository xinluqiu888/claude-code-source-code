# LocationStep.tsx — Agent位置选择步骤

> **一句话总结**：向导步骤，让用户选择Agent配置的存储位置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/wizard-steps/LocationStep.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 选择Agent配置的存储位置 |

---

## 功能概述

`LocationStep`是Agent创建向导的第一步，让用户选择新Agent的存储位置：
- Project: 存储在项目目录的`.claude/agents/`中
- Personal: 存储在用户主目录的`~/.claude/agents/`中

---

## 核心内容详解

### 主要函数

- **LocationStep**: 向导步骤组件
  - **选项**:
    - `projectSettings`: Project (.claude/agents/)
    - `userSettings`: Personal (~/.claude/agents/)
  - **操作**: 选择后自动进入下一步

### 使用向导API

- `useWizard<AgentWizardData>()`: 获取向导上下文
- `updateWizardData`: 更新位置数据
- `goNext`: 进入下一步
- `cancel`: 取消向导

### UI组件

- **WizardDialogLayout**: 向导对话框布局
  - subtitle: "Choose location"
  - footerText: 键盘快捷键提示
- **Select**: 选择器组件

---

## 设计要点

- 简单的二选一界面
- 选择后立即进入下一步
- 支持取消操作

---

## 与其他文件的关系

- **依赖**:
  - `useWizard`, `WizardDialogLayout` - 向导框架
  - `Select` - 选择组件
  - `SettingSource` - 设置来源类型
- **被依赖**:
  - `CreateAgentWizard` - 作为第1步使用
