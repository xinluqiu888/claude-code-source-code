# ModelSelector.tsx — Agent模型选择器

> **一句话总结**：提供AI模型选择界面，支持选择Agent使用的 Claude 模型。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/ModelSelector.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 为Agent选择AI模型配置 |

---

## 功能概述

`ModelSelector`组件允许用户为Agent选择使用的AI模型。它提供：
- 预定义的模型选项列表
- 支持自定义模型ID（自动注入到选项中）
- 模型选择说明提示
- 取消/完成回调支持

---

## 核心内容详解

### 主要类型

- **ModelSelectorProps**:
  - `initialModel?`: 初始选中的模型
  - `onComplete`: 完成选择回调
  - `onCancel?`: 取消回调

### 主要函数

- **ModelSelector**: 主组件函数
  - **模型选项构建**:
    - 获取基础选项列表`getAgentModelOptions()`
    - 如果initialModel不在列表中，自动添加为自定义选项
  - **默认值**: 使用initialModel或默认"sonnet"

### 渲染逻辑

1. **说明文本**: 显示"Model determines the agent's reasoning capabilities and speed."
2. **Select组件**: 渲染模型选择下拉列表
   - 传入所有模型选项
   - 设置默认值
   - 绑定onChange和onCancel回调

### 特殊处理

- **自定义模型支持**: 如果Agent当前使用的模型是完整的模型ID（如'claude-opus-4-5'）且不在别名列表中，会将其作为"Current model (custom ID)"选项注入到列表首位，确保用户确认时不会被覆盖。

---

## 设计要点

- 使用React Compiler优化渲染
- 使用现有的Select组件保持一致性
- 优雅处理自定义模型ID的情况
- 简洁的界面，只显示必要信息

---

## 与其他文件的关系

- **依赖**:
  - `getAgentModelOptions` - 获取模型选项
  - `Select` - 选择器组件
- **被依赖**:
  - `AgentEditor` - 作为模型编辑子组件

---

## 注意事项

- 如果未提供onCancel，取消操作会调用onComplete(undefined)
- 支持显示自定义模型ID，避免配置丢失
- 默认选中sonnet模型
