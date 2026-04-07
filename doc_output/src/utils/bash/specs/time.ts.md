# time.ts — time命令规格定义

> **一句话总结**：定义time命令的CommandSpec，用于计时命令执行。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/specs/time.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~14行 |
| 主要职责 | time命令的规格定义 |

---

## 功能概述
time命令用于测量命令的执行时间，输出real/user/sys时间统计。

---

## 核心内容详解

### CommandSpec定义

- **name**: `'time'`
- **description**: `'Time a command'`
- **args**: 
  - `name`: 'command' - 要计时的命令
  - `isCommand`: true - 标记为包装命令

### isCommand标记

time会包装实际执行的命令，权限系统会递归提取内部命令的前缀。

---

## 与其他文件的关系

- **被依赖**：
  - `./index.ts` - 导出到specs集合
