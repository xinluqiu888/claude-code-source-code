# nohup.ts — nohup命令规格定义

> **一句话总结**：定义nohup命令的CommandSpec，用于在忽略挂起信号的情况下运行命令。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/specs/nohup.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~14行 |
| 主要职责 | nohup命令的规格定义 |

---

## 功能概述

nohup命令用于运行免疫于挂起信号（SIGHUP）的命令，常用于后台长时间运行任务。

---

## 核心内容详解

### CommandSpec定义

- **name**: `'nohup'`
- **description**: `'Run a command immune to hangups'`
- **args**: 
  - `name`: 'command' - 要运行的命令
  - `isCommand`: true - 标记为包装命令

### isCommand标记

`isCommand: true`表示该参数是一个被包装的命令，权限系统会递归提取被包装命令的前缀。

---

## 与其他文件的关系

- **被依赖**：
  - `./index.ts` - 导出到specs集合
