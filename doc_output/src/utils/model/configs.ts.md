# configs.ts — 模型配置定义

> **一句话总结**：定义所有Claude模型在各平台（1P/Bedrock/Vertex/Foundry）的完整配置。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/model/configs.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~119行 |
| 主要职责 | 模型ID跨平台映射 |

---

## 功能概述

该模块定义了每个Claude模型在各API提供商的完整模型ID。

---

## 核心内容详解

### 模型配置

每个模型有4个平台版本：
- `firstParty`: api.anthropic.com
- `bedrock`: AWS Bedrock
- `vertex`: Google Cloud Vertex AI
- `foundry`: Azure AI Foundry

### 包含的模型

- CLAUDE_3_7_SONNET_CONFIG
- CLAUDE_3_5_V2_SONNET_CONFIG
- CLAUDE_3_5_HAIKU_CONFIG
- CLAUDE_HAIKU_4_5_CONFIG
- CLAUDE_SONNET_4_CONFIG
- CLAUDE_SONNET_4_5_CONFIG
- CLAUDE_OPUS_4_CONFIG
- CLAUDE_OPUS_4_1_CONFIG
- CLAUDE_OPUS_4_5_CONFIG
- CLAUDE_OPUS_4_6_CONFIG
- CLAUDE_SONNET_4_6_CONFIG

### 导出常量

- **`ALL_MODEL_CONFIGS`** - 所有配置的集合
- **`CANONICAL_MODEL_IDS`** - 1P格式模型ID列表
- **`CANONICAL_ID_TO_KEY`** - ID到内部键的映射

---

## 与其他文件的关系

- **被依赖**：
  - modelStrings.ts - 获取平台特定模型字符串

---

## 注意事项

- 新模型需要在此处注册
- 模型启动时需更新tengu_ant_model_override
- 添加模型后更新CANONICAL_ID_TO_KEY映射
