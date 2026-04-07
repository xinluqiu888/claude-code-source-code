# timeout.ts — timeout命令规格定义

> **一句话总结**：定义timeout命令的CommandSpec，用于在指定时间限制内运行命令。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/specs/timeout.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~21行 |
| 主要职责 | timeout命令的规格定义 |

---

## 功能概述

timeout命令用于在指定时间后终止命令执行，常用于防止脚本挂起。

---

## 核心内容详解

### CommandSpec定义

- **name**: `'timeout'`
- **description**: `'Run a command with a time limit'`

### Args（数组）

| 位置 | 名称 | 描述 | 可选 |
|------|------|------|------|
| 1 | duration | 超时时间（如10, 5s, 2m） | 否 |
| 2 | command | 要运行的命令 | 否 |

### isCommand标记

第二个参数（command）标记为`isCommand: true`，权限系统会递归提取被包装命令。

---

## 与其他文件的关系

- **被依赖**：
  - `./index.ts` - 导出到specs集合
