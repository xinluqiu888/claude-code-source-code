# validateAgent.ts — Agent配置验证器

> **一句话总结**：验证Agent配置的完整性和有效性。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/validateAgent.ts` |
| 文件类型 | TypeScript模块 |
| 主要职责 | 验证Agent定义的各项配置是否合规 |

---

## 功能概述

`validateAgent.ts`提供Agent配置的验证功能，确保：
- Agent标识符符合命名规范
- 描述信息充足且长度合适
- 工具配置有效
- 系统提示词完整
- 避免重复定义

---

## 核心内容详解

### 导入与依赖

- `resolveAgentTools`: 解析Agent工具配置
- `AgentDefinition`, `CustomAgentDefinition`: Agent类型定义
- `getAgentSourceDisplayName`: 获取来源显示名称

### 主要类型

- **AgentValidationResult**: 验证结果
  - `isValid`: 是否有效（无错误）
  - `errors`: 错误信息数组
  - `warnings`: 警告信息数组

### 主要函数

- **validateAgentType**: 验证Agent类型名称
  - 必须非空
  - 必须以字母数字开头和结尾
  - 只能包含字母、数字和连字符
  - 长度3-50字符
  - **返回**: 错误信息或null

- **validateAgent**: 验证完整Agent配置
  - **参数**: agent, availableTools, existingAgents
  - **验证项**:
    1. **Agent类型**:
       - 非空检查
       - 格式验证（调用validateAgentType）
       - 重复检查（排除自身）
    
    2. **描述（whenToUse）**:
       - 非空检查
       - 长度警告（<10字符太短，>5000字符太长）
    
    3. **工具**:
       - 必须是数组类型
       - undefined表示所有工具（警告）
       - 空数组警告
       - 无效工具检查（调用resolveAgentTools）
    
    4. **系统提示词**:
       - 非空检查
       - 长度检查（<20字符错误，>10000字符警告）
  - **返回**: AgentValidationResult对象

---

## 设计要点

- 区分错误（阻止保存）和警告（提醒用户）
- 详细的命名规范验证
- 智能的重复检测（编辑时排除自身）
- 工具有效性验证使用实际解析逻辑

---

## 与其他文件的关系

- **依赖**:
  - `resolveAgentTools` - 工具解析验证
  - `getAgentSourceDisplayName` - 来源名称显示
  - `AgentDefinition` - 类型定义
- **被依赖**:
  - `CreateAgentWizard` - 创建时验证
  - `ConfirmStepWrapper` - 确认步骤验证

---

## 注意事项

- 编辑现有Agent时，重复检查需要排除自身
- 工具undefined和空数组是不同的语义
- 警告不会阻止保存，但会提醒用户
- 标识符限制为ASCII字符集
