# gitConfigParser.ts — Git配置文件解析器

> **一句话总结**：轻量级解析.git/config文件，提取指定section/subsection/key的配置值。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/git/gitConfigParser.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~278行 |
| 主要职责 | 解析Git配置文件格式 |

---

## 功能概述

该模块提供对Git配置文件的解析功能，遵循git config.c的解析规则：
- Section名：不区分大小写，字母数字+连字符
- Subsection名（引号内）：区分大小写，支持反斜杠转义
- Key名：不区分大小写，字母数字+连字符
- Value：可选引号，支持内联注释和反斜杠转义

---

## 核心内容详解

### 主要导出函数

- **`parseGitConfigValue`**
  - 参数：`gitDir`（string）, `section`（string）, `subsection`（string|null）, `key`（string）
  - 返回值：`Promise<string | null>`
  - 用途：从.git/config文件读取指定配置值

- **`parseConfigString`**
  - 参数：`config`（string）, `section`, `subsection`, `key`
  - 返回值：`string | null`
  - 用途：从内存中的配置字符串解析值

### 内部函数

- **`parseKeyValue`** - 解析key = value行
- **`parseValue`** - 解析配置值，处理引号和转义
- **`matchesSectionHeader`** - 匹配section头部（如[remote "origin"]）
- **`isKeyChar`** - 验证key名字符

### 转义序列支持

| 序列 | 结果 |
|------|------|
| `\n` | 换行 |
| `\t` | Tab |
| `\b` | 退格 |
| `\"` | 双引号 |
| `\\` | 反斜杠 |

---

## 与其他文件的关系

- **依赖**：
  - `fs/promises` - 文件读取
  - `path` - 路径拼接
- **被依赖**：
  - `gitFilesystem.ts` - 读取remote url等配置

---

## 注意事项

- 与git config命令行为一致
- 不处理多行值（单行解析器）
- 忽略布尔键（无值的key）
