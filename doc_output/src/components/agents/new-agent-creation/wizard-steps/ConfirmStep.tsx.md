# ConfirmStep.tsx — Agent创建确认步骤

> **一句话总结**：显示Agent配置摘要，验证配置并确认保存。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/wizard-steps/ConfirmStep.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 显示配置摘要、验证和保存确认 |

---

## 功能概述

`ConfirmStep`显示Agent配置的摘要信息，让用户确认配置无误后保存：
- 显示Agent类型、描述、工具、模型、颜色等配置
- 验证Agent配置的有效性
- 显示验证错误和警告
- 提供保存和保存后编辑选项

---

## 核心内容详解

### Props

- **tools**: 可用工具集
- **existingAgents**: 现有Agent列表
- **onSave**: 保存回调
- **onSaveAndEdit**: 保存并打开编辑器回调
- **error?**: 保存错误信息

### 主要函数

- **ConfirmStep**: 确认步骤组件
  - **验证**: 调用`validateAgent`验证配置
  - **系统提示词预览**: 截断显示前240字符
  - **键盘处理**: 
    - `s` 或 Enter: 保存
    - `e`: 保存并编辑
    - Esc: 返回上一步

### 显示内容

1. **文件路径**: 显示Agent文件的存储位置
2. **Agent类型**: 标识符名称
3. **描述**: whenToUse内容
4. **工具**: 显示选中工具数量或"All tools"
5. **模型**: 显示选中的模型（如有）
6. **颜色**: 显示选中的颜色（如有）
7. **内存**: 显示内存范围（如启用）
8. **系统提示词**: 显示前240字符预览

### 验证显示

- **错误**: 红色显示，阻止保存
- **警告**: 黄色显示，不阻止保存

---

## 设计要点

- 在保存前进行完整的配置验证
- 系统提示词过长时截断显示
- 清晰的保存选项（直接保存 vs 保存并编辑）

---

## 与其他文件的关系

- **依赖**:
  - `useWizard`, `WizardDialogLayout` - 向导框架
  - `validateAgent` - 验证函数
  - `getNewRelativeAgentFilePath` - 获取文件路径
  - `getAgentModelDisplay` - 模型显示
  - `getMemoryScopeDisplay` - 内存范围显示
  - `truncateToWidth` - 文本截断
- **被依赖**:
  - `ConfirmStepWrapper` - 作为UI组件使用
