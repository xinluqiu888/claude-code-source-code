# antModels.ts — Anthropic内部模型配置

> **一句话总结**：管理Anthropic内部员工的模型覆盖配置（特性门控）。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/model/antModels.ts` |
| 文件行数 | ~65行 |
| 主要职责 | Ant-only模型配置管理 |

---

## 功能概述

该模块为Anthropic内部员工提供模型覆盖配置，通过Growthbook特性开关管理。

---

## 核心内容详解

### 导出类型

- **`AntModel`** - 内部模型定义
- **`AntModelOverrideConfig`** - 覆盖配置结构

### 导出函数

- **`getAntModelOverrideConfig`**
  - 用途：获取ant-only模型覆盖配置
  - 条件：USER_TYPE === 'ant'

- **`getAntModels`**
  - 用途：获取内部模型列表

- **`resolveAntModel`**
  - 用途：根据别名或模型名解析内部模型

---

## 与其他文件的关系

- **依赖**：
  - `src/services/analytics/growthbook.ts` - 特性开关
- **被依赖**：
  - model.ts等模型选择模块

---

## 注意事项

- 仅在USER_TYPE=ant时可用
- 内部模型代号需要添加到excluded-strings.txt防止泄露
