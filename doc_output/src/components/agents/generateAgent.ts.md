# generateAgent.ts — Agent自动生成器

> **一句话总结**：使用AI模型根据用户描述自动生成Agent配置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/generateAgent.ts` |
| 文件类型 | TypeScript模块 |
| 主要职责 | 通过AI生成Agent的标识符、描述和系统提示词 |

---

## 功能概述

`generateAgent.ts`利用Claude AI的能力，根据用户的自然语言描述自动生成完整的Agent配置。它：
- 分析用户意图和需求
- 生成合适的Agent标识符
- 编写使用场景描述（whenToUse）
- 创建专业的系统提示词
- 可选添加内存管理指令

---

## 核心内容详解

### 导入与依赖

- `queryModelWithoutStreaming`: 非流式AI查询
- `getUserContext`: 获取用户上下文
- `createUserMessage`, `normalizeMessagesForAPI`: 消息处理
- `isAutoMemoryEnabled`: 检查内存功能是否启用

### 主要类型

- **GeneratedAgent**: 生成的Agent结构
  - `identifier`: Agent标识符
  - `whenToUse`: 使用场景描述
  - `systemPrompt`: 系统提示词

### 系统提示词

- **AGENT_CREATION_SYSTEM_PROMPT**: 指导AI生成Agent的系统提示
  - 提取核心意图和需求
  - 设计专家角色
  - 构建全面的系统指令
  - 优化性能机制
  - 创建标识符（小写字母、数字、连字符）
  - 包含使用示例

- **AGENT_MEMORY_INSTRUCTIONS**: 内存管理指令模板
  - 当启用内存功能时追加到系统提示
  - 提供特定领域的内存更新指导

### 主要函数

- **generateAgent**: 生成Agent的主函数
  - **参数**: userPrompt, model, existingIdentifiers, abortSignal
  - **流程**:
    1. 构建提示词，包含已有标识符列表避免重复
    2. 获取用户上下文
    3. 根据内存功能状态选择系统提示词
    4. 调用AI模型生成配置
    5. 解析JSON响应
    6. 验证返回结构完整性
    7. 记录分析事件
  - **返回**: GeneratedAgent对象

### 解析逻辑

- 尝试直接解析整个响应文本为JSON
- 如果失败，尝试从响应中提取JSON对象
- 验证必需字段（identifier, whenToUse, systemPrompt）
- 失败时抛出错误

---

## 设计要点

- 使用系统提示词精确控制AI生成行为
- 支持上下文感知（CLAUDE.md等）
- 避免标识符重复
- 可选的内存管理指令集成
- 详细的示例指导确保输出质量

---

## 与其他文件的关系

- **依赖**:
  - `queryModelWithoutStreaming` - AI查询
  - `getUserContext`, `prependUserContext` - 上下文管理
  - `isAutoMemoryEnabled` - 功能开关
  - `logEvent` - 分析记录
- **被依赖**:
  - `GenerateStep` - 在创建向导中调用

---

## 注意事项

- 生成的标识符必须唯一（通过existingIdentifiers检查）
- 返回必须是有效的JSON格式
- 系统提示词会包含项目特定的CLAUDE.md上下文
- 内存指令仅在GrowthBook功能启用时包含
- 使用thinking disabled模式加快响应
