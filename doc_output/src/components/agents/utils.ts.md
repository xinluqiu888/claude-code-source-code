# utils.ts — Agent工具函数

> **一句话总结**：提供Agent相关的通用工具函数。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/utils.ts` |
| 文件类型 | TypeScript模块 |
| 主要职责 | 提供Agent来源的显示名称转换 |

---

## 功能概述

`utils.ts`提供Agent模块使用的通用工具函数，目前主要包含Agent来源的显示名称转换功能。

---

## 核心内容详解

### 导入与依赖

- `capitalize`: 字符串首字母大写
- `SettingSource`: 设置来源类型
- `getSettingSourceName`: 获取来源内部名称

### 主要函数

- **getAgentSourceDisplayName**: 获取Agent来源的显示名称
  - **参数**: source (SettingSource | 'all' | 'built-in' | 'plugin')
  - **返回值**: 人类可读的来源名称
  - **映射**:
    - 'all' -> 'Agents'
    - 'built-in' -> 'Built-in agents'
    - 'plugin' -> 'Plugin agents'
    - 其他 -> 首字母大写的来源名称

---

## 设计要点

- 简单的映射逻辑，易于扩展
- 统一的显示名称格式化
- 复用现有的capitalize和getSettingSourceName

---

## 与其他文件的关系

- **依赖**:
  - `capitalize` - 字符串工具
  - `getSettingSourceName` - 来源名称工具
- **被依赖**:
  - `AgentsList` - 显示来源分组标题
  - `validateAgent.ts` - 显示重复来源信息

---

## 注意事项

- 特殊来源（all、built-in、plugin）有固定显示名称
- 其他来源使用首字母大写格式化
