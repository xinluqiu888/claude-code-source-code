# types.ts — 内存类型定义

> **一句话总结**：定义内存（Memory）的类型系统和常量值。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/memory/types.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~13行 |
| 主要职责 | 内存类型系统的类型定义 |

---

## 功能概述

该模块定义了Claude Code中"Memory"功能的类型系统。Memory用于持久化存储用户、项目或团队级别的上下文信息。

---

## 核心内容详解

### 导出常量

- **`MEMORY_TYPE_VALUES`**
  - 类型：字符串数组
  - 基础值：`['User', 'Project', 'Local', 'Managed', 'AutoMem']`
  - 条件值：当`feature('TEAMMEM')`启用时添加`'TeamMem'`

### 导出类型

- **`MemoryType`**
  - 类型：联合类型
  - 值：从MEMORY_TYPE_VALUES推导

### 内存类型说明

| 类型 | 用途 |
|------|------|
| `User` | 用户级全局内存 |
| `Project` | 项目级内存（绑定到git仓库） |
| `Local` | 本地会话内存 |
| `Managed` | 托管内存（系统管理） |
| `AutoMem` | 自动生成的内存 |
| `TeamMem` | 团队共享内存（特性门控） |

---

## 与其他文件的关系

- **依赖**：
  - `bun:bundle`的`feature`函数
- **被依赖**：
  - 所有Memory相关模块
