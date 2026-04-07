# aliases.ts — 模型别名定义

> **一句话总结**：定义所有支持的模型别名及其类型。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/model/aliases.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~26行 |
| 主要职责 | 模型别名常量和类型定义 |

---

## 功能概述

该模块定义了用户可用的模型别名，作为完整模型ID的简写。

---

## 核心内容详解

### 导出常量

- **`MODEL_ALIASES`**
  - 值：`['sonnet', 'opus', 'haiku', 'best', 'sonnet[1m]', 'opus[1m]', 'opusplan']`
  - 说明：用户可输入的模型别名列表

- **`MODEL_FAMILY_ALIASES`**
  - 值：`['sonnet', 'opus', 'haiku']`
  - 说明：模型系列别名（允许列表中的通配符）

### 导出类型和函数

- **`ModelAlias`** - 从MODEL_ALIASES推导的联合类型
- **`isModelAlias`** - 检查字符串是否为模型别名
- **`isModelFamilyAlias`** - 检查是否为系列别名

---

## 与其他文件的关系

- **被依赖**：
  - 所有模型选择相关模块
