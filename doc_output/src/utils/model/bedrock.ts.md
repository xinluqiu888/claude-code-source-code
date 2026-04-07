# bedrock.ts — AWS Bedrock集成

> **一句话总结**：提供AWS Bedrock推理配置和区域前缀管理。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/model/bedrock.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~266行 |
| 主要职责 | Bedrock客户端和配置管理 |

---

## 功能概述

该模块处理与AWS Bedrock的集成：
1. 创建Bedrock客户端（带代理和认证）
2. 列出推理配置文件
3. 提取和管理区域前缀（us/eu/apac/global）
4. ARN到模型ID的转换

---

## 核心内容详解

### 导出函数

- **`getBedrockInferenceProfiles`** - 列出可用推理配置文件
- **`createBedrockClient`** - 创建Bedrock管理客户端
- **`createBedrockRuntimeClient`** - 创建Bedrock运行时客户端
- **`getInferenceProfileBackingModel`** - 获取配置文件的后端模型
- **`extractModelIdFromArn`** - 从ARN提取模型ID
- **`getBedrockRegionPrefix`** / **`applyBedrockRegionPrefix`** - 区域前缀管理

### 区域前缀

支持：`us`, `eu`, `apac`, `global`

用于跨区域推理（cross-region inference）。

---

## 与其他文件的关系

- **依赖**：
  - `@aws-sdk/client-bedrock`
  - `@aws-sdk/client-bedrock-runtime`
  - `../auth.ts` - AWS凭证
  - `../proxy.ts` - 代理配置
- **被依赖**：
  - model.ts, modelStrings.ts等

---

## 注意事项

- 支持CLAUDE_CODE_SKIP_BEDROCK_AUTH跳过认证
- 支持ANTHROPIC_BEDROCK_BASE_URL自定义端点
- 区域前缀影响数据驻留和延迟
