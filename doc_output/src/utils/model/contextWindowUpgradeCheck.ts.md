# contextWindowUpgradeCheck.ts — 上下文窗口升级提示

> **一句话总结**：检查并向用户提示可用的1M上下文模型升级。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/model/contextWindowUpgradeCheck.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~48行 |
| 主要职责 | 上下文升级可用性检查 |

---

## 功能概述

该模块在用户当前模型接近上下文限制时，提示可用的1M上下文升级选项。

---

## 核心内容详解

### 导出函数

- **`getUpgradeMessage`**
  - 参数：`context`（'warning' | 'tip'）
  - 返回值：string | null
  - 用途：生成升级提示消息

### 升级选项

- `opus[1m]` - Opus 1M上下文（5x容量）
- `sonnet[1m]` - Sonnet 1M上下文（5x容量）

---

## 与其他文件的关系

- **依赖**：
  - `./check1mAccess.ts` - 访问权限检查
  - `./model.ts` - 获取用户当前模型
- **被依赖**：
  - UI提示系统

---

## 注意事项

- 仅在用户有访问权限且有可用升级时返回消息
- 两种上下文：warning（警告时）和tip（提示）
