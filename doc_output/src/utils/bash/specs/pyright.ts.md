# pyright.ts — Pyright命令规格定义

> **一句话总结**：提供Pyright类型检查器的完整CommandSpec，包含所有子命令、选项和参数定义。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/specs/pyright.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~92行 |
| 主要职责 | Pyright命令的详细规格定义 |

---

## 功能概述

Pyright是Python的静态类型检查器，该spec定义了其完整的CLI接口，用于命令补全和参数分析。

---

## 核心内容详解

### CommandSpec定义

- **name**: `'pyright'`
- **description**: `'Type checker for Python'`

### Options（部分列举）

| 选项 | 描述 |
|------|------|
| `--help, -h` | 显示帮助 |
| `--version` | 显示版本 |
| `--watch, -w` | 持续运行并监视变更 |
| `--project, -p` | 指定配置文件位置 |
| `--createstub` | 创建类型存根文件 |
| `--verifytypes` | 验证py.typed包的类型完整性 |
| `--pythonpath` | Python解释器路径 |
| `--outputjson` | JSON格式输出 |

### Args

- **name**: 'files'
- **description**: 要分析的文件或目录
- **isVariadic**: true - 可接受多个文件
- **isOptional**: true - 可选

---

## 与其他文件的关系

- **被依赖**：
  - `./index.ts` - 导出到specs集合
