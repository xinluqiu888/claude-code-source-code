# GenerateStep.tsx — AI生成Agent步骤

> **一句话总结**：接收用户描述并使用AI自动生成Agent配置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/new-agent-creation/wizard-steps/GenerateStep.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 使用AI根据描述生成Agent配置 |

---

## 功能概述

`GenerateStep`接收用户的自然语言描述，调用AI生成完整的Agent配置：
- 接收用户的Agent功能描述
- 调用`generateAgent`函数生成配置
- 显示生成进度和错误信息
- 支持取消生成过程
- 生成成功后自动跳转到工具配置步骤

---

## 核心内容详解

### 主要函数

- **GenerateStep**: 向导步骤组件
  - **状态**:
    - `prompt`: 用户输入的描述
    - `isGenerating`: 是否正在生成
    - `error`: 错误信息
    - `cursorOffset`: 光标位置

- **handleGenerate**: 处理生成请求
  - 验证输入非空
  - 创建AbortController支持取消
  - 调用`generateAgent`生成配置
  - 更新wizardData包含生成的配置
  - 成功后跳转到ToolsStep (index 6)

- **handleCancelGeneration**: 取消生成
  - 调用AbortController.abort()
  - 显示取消状态

- **handleGoBack**: 返回处理
  - 清空所有生成相关数据
  - 返回上一步

- **handleExternalEditor**: 在外部编辑器中编辑
  - 支持Ctrl+G快捷键
  - 打开系统默认编辑器

### 生成流程

1. 用户输入描述
2. 点击Enter提交
3. 显示生成中状态（Spinner）
4. 调用AI生成Agent配置
5. 成功后更新wizardData并跳转
6. 失败显示错误信息

### 错误处理

- 输入为空错误
- 生成失败错误
- 用户取消（不显示错误）
- API错误处理

---

## 设计要点

- 支持取消正在进行的生成
- 提供外部编辑器支持长文本输入
- 生成成功后直接跳转到工具步骤
- 使用AbortController支持取消操作

---

## 与其他文件的关系

- **依赖**:
  - `useWizard`, `WizardDialogLayout` - 向导框架
  - `TextInput` - 文本输入
  - `Spinner` - 加载指示器
  - `generateAgent` - AI生成函数
  - `useMainLoopModel` - 获取当前模型
  - `editPromptInEditor` - 外部编辑器
- **被依赖**:
  - `CreateAgentWizard` - 作为第3步使用（AI生成模式）
