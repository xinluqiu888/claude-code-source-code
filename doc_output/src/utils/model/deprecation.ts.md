# deprecation.ts — 模型弃用管理

> **一句话总结**：跟踪已弃用模型的退役日期，向用户显示弃用警告。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/model/deprecation.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~102行 |
| 主要职责 | 模型弃用信息管理和提示 |

---

## 功能概述

该模块维护已弃用模型的信息，并在用户使用这些模型时显示警告。

---

## 核心内容详解

### 弃用模型列表

- `claude-3-opus` - 2026年1月退役
- `claude-3-7-sonnet` - 2026年2-5月退役
- `claude-3-5-haiku` - 2026年2月退役（仅1P）

### 导出函数

- **`getModelDeprecationWarning`**
  - 参数：`modelId`（string | null）
  - 返回值：string | null
  - 用途：获取模型的弃用警告消息

---

## 与其他文件的关系

- **依赖**：
  - `./providers.ts` - 获取API提供商
- **被依赖**：
  - 模型选择UI

---

## 注意事项

- 退役日期按平台不同（1P/Bedrock/Vertex/Foundry）
- null表示该提供商不弃用
- 按子串匹配模型ID
