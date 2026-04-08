# new-agent-creation/ — 新Agent创建向导

> **一句话总结**：包含Agent创建向导的所有步骤组件和类型定义。

---

## 目录职责

`new-agent-creation/`目录包含使用向导模式创建新Agent的所有步骤组件。向导引导用户逐步完成Agent配置：
- 选择存储位置
- 选择创建方式（AI生成或手动）
- 配置Agent属性（类型、描述、提示词、工具、模型、颜色、内存）
- 确认并保存

---

## 文件清单

### 主入口

| 文件 | 职责简述 |
|------|---------|
| [CreateAgentWizard.tsx](./CreateAgentWizard.tsx.md) | 向导主组件，协调所有步骤 |

### 向导步骤

| 文件 | 职责简述 |
|------|---------|
| [LocationStep.tsx](./wizard-steps/LocationStep.tsx.md) | 选择Agent存储位置 |
| [MethodStep.tsx](./wizard-steps/MethodStep.tsx.md) | 选择创建方式（AI生成/手动） |
| [GenerateStep.tsx](./wizard-steps/GenerateStep.tsx.md) | AI自动生成Agent配置 |
| [TypeStep.tsx](./wizard-steps/TypeStep.tsx.md) | 输入Agent类型标识符 |
| [PromptStep.tsx](./wizard-steps/PromptStep.tsx.md) | 输入系统提示词 |
| [DescriptionStep.tsx](./wizard-steps/DescriptionStep.tsx.md) | 输入使用场景描述 |
| [ToolsStep.tsx](./wizard-steps/ToolsStep.tsx.md) | 选择可用工具 |
| [ModelStep.tsx](./wizard-steps/ModelStep.tsx.md) | 选择AI模型 |
| [ColorStep.tsx](./wizard-steps/ColorStep.tsx.md) | 选择颜色标识 |
| [MemoryStep.tsx](./wizard-steps/MemoryStep.tsx.md) | 配置内存范围（条件显示） |
| [ConfirmStep.tsx](./wizard-steps/ConfirmStep.tsx.md) | 显示配置摘要并确认 |
| [ConfirmStepWrapper.tsx](./wizard-steps/ConfirmStepWrapper.tsx.md) | 处理保存逻辑 |

---

## 向导流程

```
LocationStep -> MethodStep -> [GenerateStep] -> TypeStep -> PromptStep -> 
DescriptionStep -> ToolsStep -> ModelStep -> ColorStep -> [MemoryStep] -> 
ConfirmStep/ConfirmStepWrapper
```

- `[]` 表示条件步骤
- GenerateStep仅在AI生成模式下显示
- MemoryStep仅在GrowthBook功能启用时显示

---

## 数据流

1. 每个步骤通过`useWizard`获取和更新wizardData
2. 步骤数据在AgentWizardData类型中定义
3. ConfirmStepWrapper整合所有数据并保存到文件系统

---

## 设计要点

- 使用WizardProvider框架管理步骤状态
- 每个步骤独立负责特定配置项
- 支持动态步骤（根据选择跳过某些步骤）
- 条件性包含内存步骤
