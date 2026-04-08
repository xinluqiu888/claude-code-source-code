# MemoryStep.tsx — Agent内存配置步骤

> **一句话总结**：配置Agent的持久化内存范围。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/wizard-steps/MemoryStep.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 配置Agent的记忆存储范围 |

---

## 功能概述

`MemoryStep`让用户选择Agent的持久化内存范围，支持：
- User scope: 用户级存储
- Project scope: 项目级存储
- Local scope: 本地存储
- None: 无持久化内存

---

## 核心内容详解

### 主要类型

- **MemoryOption**: 内存选项
  - `label`: 显示文本
  - `value`: AgentMemoryScope | 'none'

### 主要函数

- **MemoryStep**: 向导步骤组件
  - **动态选项**: 根据Agent存储位置推荐默认选项
    - 用户级Agent默认推荐user scope
    - 项目级Agent默认推荐project scope
  - **处理选择**:
    - 保存选中的内存范围
    - 如启用内存，在系统提示词中追加内存加载指令
    - 更新finalAgent

### 内存范围选项

- **user**: ~/.claude/agent-memory/
- **project**: .claude/agent-memory/
- **local**: .claude/agent-memory-local/
- **none**: 无持久化内存

### 系统提示词增强

如果启用内存且Agent类型存在：
- 使用`isAutoMemoryEnabled()`检查功能开关
- 使用`loadAgentMemoryPrompt()`生成内存加载指令
- 追加到系统提示词中

---

## 设计要点

- 根据Agent位置智能推荐内存范围
- 动态更新系统提示词包含内存指令
- 支持条件性包含（功能开关控制）

---

## 与其他文件的关系

- **依赖**:
  - `useWizard`, `WizardDialogLayout` - 向导框架
  - `Select` - 选择组件
  - `isAutoMemoryEnabled`, `loadAgentMemoryPrompt` - 内存功能
- **被依赖**:
  - `CreateAgentWizard` - 条件性作为内存步骤使用
