# CreateAgentWizard.tsx — Agent创建向导

> **一句话总结**：多步骤向导组件，引导用户完成新Agent的创建流程。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/CreateAgentWizard.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 协调Agent创建向导的各个步骤 |

---

## 功能概述

`CreateAgentWizard`是Agent创建的入口组件，它使用`WizardProvider`框架构建一个多步骤向导：
- 步骤包括：位置选择、创建方式、AI生成、类型定义、提示词编写、描述、工具、模型、颜色、内存、确认
- 支持手动创建和AI自动生成两种方式
- 条件性包含内存步骤（基于GrowthBook功能开关）

---

## 核心内容详解

### 导入的步骤组件

- **LocationStep**: 选择Agent存储位置（用户/项目/策略设置）
- **MethodStep**: 选择创建方式（手动或AI生成）
- **GenerateStep**: AI自动生成Agent配置
- **TypeStep**: 输入Agent类型标识符
- **PromptStep**: 编写系统提示词
- **DescriptionStep**: 编写使用场景描述
- **ToolsStep**: 选择可用工具
- **ModelStep**: 选择AI模型
- **ColorStep**: 选择颜色标识
- **MemoryStep**: 配置内存范围（条件显示）
- **ConfirmStepWrapper**: 确认和保存步骤

### 主要类型

- **Props**:
  - `tools`: 可用工具集
  - `existingAgents`: 现有Agent列表
  - `onComplete`: 完成回调（携带成功消息）
  - `onCancel`: 取消回调

### 主要函数

- **CreateAgentWizard**: 主组件函数
  - **步骤构建**:
    1. LocationStep - 位置选择
    2. MethodStep - 创建方式
    3. GenerateStep - AI生成（可选）
    4. TypeStep - 类型标识符
    5. PromptStep - 系统提示词
    6. DescriptionStep - 使用描述
    7. ToolsStep - 工具配置
    8. ModelStep - 模型选择
    9. ColorStep - 颜色选择
    10. MemoryStep - 内存配置（条件）
    11. ConfirmStepWrapper - 确认保存
  
  - **条件逻辑**: 
    - MemoryStep仅在`isAutoMemoryEnabled()`为true时包含

  - **WizardProvider配置**:
    - 标题: "Create new agent"
    - 不显示步骤计数器
    - 空初始数据
    - 完成和取消回调

---

## 设计要点

- 使用WizardProvider统一管理步骤流程
- 某些步骤使用工厂函数传递props
- 条件步骤根据功能开关动态包含
- 确认步骤包装器处理实际的保存逻辑

---

## 与其他文件的关系

- **依赖**:
  - `WizardProvider` - 向导框架
  - 所有wizard-steps组件
  - `isAutoMemoryEnabled` - 功能开关
- **被依赖**:
  - `AgentsMenu` - 在create-agent模式下渲染

---

## 注意事项

- 步骤顺序是固定的，不能动态重排
- TypeStep、ToolsStep、ConfirmStepWrapper使用工厂函数以传递props
- 向导完成时的实际逻辑由ConfirmStepWrapper处理
- 内存步骤可能不出现，取决于功能开关状态
