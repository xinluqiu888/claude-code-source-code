# shellQuoting.ts — Shell命令引用工具

> **一句话总结**：提供高级Shell命令引用功能，支持Heredoc和多行字符串的特殊处理。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/shellQuoting.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~129行 |
| 主要职责 | 高级Shell命令引用和Windows重定向符转换 |

---

## 功能概述

该模块在shellQuote.ts基础上提供更高级的命令处理：
1. **Heredoc检测**：识别`<<EOF`语法，避免添加stdin重定向
2. **多行字符串检测**：识别跨行引用的字符串
3. **Windows重定向符转换**：将`>nul`转换为`/dev/null`
4. **智能stdin重定向**：判断是否需要添加`< /dev/null`

---

## 核心内容详解

### 导出函数

- **`quoteShellCommand`**
  - 参数：`command`（string）, `addStdinRedirect`（boolean, 默认true）
  - 返回值：`string` - 引用的命令
  - 特殊处理：Heredoc和多行字符串使用单引号方案

- **`hasStdinRedirect`**
  - 参数：`command`（string）
  - 返回值：`boolean`
  - 检测：是否存在`< file`模式（非`<<` heredoc）

- **`shouldAddStdinRedirect`**
  - 参数：`command`（string）
  - 返回值：`boolean`
  - 逻辑：无Heredoc、无现有重定向时返回true

- **`rewriteWindowsNullRedirect`**
  - 参数：`command`（string）
  - 返回值：`string`
  - 转换：`>nul`, `2>nul`, `&>nul` → `/dev/null`

---

## 与其他文件的关系

- **依赖**：
  - `./shellQuote.ts` - 基础引用功能
- **被依赖**：
  - `../Shell.ts` - 命令执行准备

---

## 注意事项

- Windows NUL重定向是常见问题（issue #4928）
- Git Bash中`2>nul`会创建名为"nul"的文件
- Heredoc命令不应添加stdin重定向
- 多行字符串保留换行符处理
