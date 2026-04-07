# mappers.ts — 消息类型映射器

> **一句话总结**：在内部消息格式与SDK消息格式之间进行双向转换。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/messages/mappers.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~291行 |
| 主要职责 | 消息格式转换和规范化 |

---

## 功能概述

该模块提供Claude Code内部消息格式与SDK消息格式之间的转换：
- `toInternalMessages`：SDK消息 → 内部消息
- `toSDKMessages`：内部消息 → SDK消息

处理多种消息类型：assistant、user、system，以及特殊子类型如compact_boundary。

---

## 核心内容详解

### 导出函数

- **`toInternalMessages`**
  - 参数：`messages`（SDKMessage[]）
  - 返回值：`Message[]`
  - 转换：
    - assistant → 内部AssistantMessage
    - user → 内部UserMessage
    - system/subtype=compact_boundary → 内部SystemMessage

- **`toSDKMessages`**
  - 参数：`messages`（Message[]）
  - 返回值：`SDKMessage[]`
  - 转换：
    - 内部消息 → SDK消息格式
    - 处理local_command输出特殊逻辑

- **`localCommandOutputToSDKAssistantMessage`**
  - 用途：将本地命令输出（如/voice、/cost）转换为SDKAssistantMessage
  - 原因：system/local_command_output子类型不被移动端SDK识别
  - 处理：去除ANSI、解包XML标签

- **`toSDKRateLimitInfo`** / **`fromSDKCompactMetadata`**
  - 用途：SDK与内部类型的辅助转换

### 规范化处理

- **`normalizeAssistantMessageForSDK`**
  - 特殊处理ExitPlanModeV2工具的plan输入
  - 从文件读取plan内容注入到tool_input

---

## 与其他文件的关系

- **依赖**：
  - `../messages.ts` - 内部消息类型
  - `../plans.ts` - 获取plan内容
  - `strip-ansi` - 去除ANSI序列
- **被依赖**：
  - SDK桥接层
  - 消息存储和恢复

---

## 注意事项

- local_command消息需要特殊处理防止泄露到RC web UI
- 转换过程可能涉及XML解包
- compact_metadata有双向转换需求
