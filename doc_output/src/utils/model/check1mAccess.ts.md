# check1mAccess.ts — 1M上下文访问检查

> **一句话总结**：检查用户是否有权使用1M上下文的Opus和Sonnet模型。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/model/check1mAccess.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~73行 |
| 主要职责 | 1M上下文模型访问控制 |

---

## 功能概述

该模块实现1M上下文模型的访问控制逻辑：
- 检查用户是否被允许使用opus[1m]和sonnet[1m]
- 基于订阅类型和extra usage配置

---

## 核心内容详解

### 导出函数

- **`checkOpus1mAccess`** - 检查Opus 1M访问权限
- **`checkSonnet1mAccess`** - 检查Sonnet 1M访问权限

### 权限逻辑

1. 如果1M上下文被全局禁用 → false
2. Claude AI订阅者 → 检查extra usage是否启用
3. 非订阅者（API/PAYG）→ 允许访问

---

## 与其他文件的关系

- **依赖**：
  - `../auth.ts` - 订阅类型检测
  - `../config.ts` - extra usage禁用原因
  - `../context.ts` - 1M上下文全局开关
- **被依赖**：
  - model.ts等模型选择模块

---

## 注意事项

- out_of_credits仍视为启用（可欠费）
- 多种原因会导致禁用（组织级、座位级等）
