# sleep.ts — sleep命令规格定义

> **一句话总结**：定义sleep命令的CommandSpec，用于延迟指定时间。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/specs/sleep.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~14行 |
| 主要职责 | sleep命令的规格定义 |

---

## 功能概述

sleep命令用于暂停执行指定的时间，常用于脚本中的延迟。

---

## 核心内容详解

### CommandSpec定义

- **name**: `'sleep'`
- **description**: `'Delay for a specified amount of time'`
- **args**: 
  - `name`: 'duration' - 延迟时间
  - `isOptional`: false - 必需参数

### 时间格式

支持秒数或带后缀的时间：
- `5` - 5秒
- `5s` - 5秒
- `2m` - 2分钟
- `1h` - 1小时

---

## 与其他文件的关系

- **被依赖**：
  - `./index.ts` - 导出到specs集合
