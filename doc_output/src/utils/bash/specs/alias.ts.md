# alias.ts — alias命令规格定义

> **一句话总结**：定义alias命令的CommandSpec，支持创建和管理命令别名。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/specs/alias.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~15行 |
| 主要职责 | alias命令的规格定义 |

---

## 功能概述

该文件为Shell的`alias`命令提供规格定义，用于：
1. 命令补全时识别alias的参数
2. 权限系统识别alias命令的结构

---

## 核心内容详解

### CommandSpec定义

- **name**: `'alias'`
- **description**: `'Create or list command aliases'`
- **args**: 
  - `name`: 'definition' - 别名定义（name=value形式）
  - `isOptional`: true - 可选参数
  - `isVariadic`: true - 可接受多个定义

---

## 与其他文件的关系

- **被依赖**：
  - `./index.ts` - 导出到specs集合
