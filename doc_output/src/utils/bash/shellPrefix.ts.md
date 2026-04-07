# shellPrefix.ts — Shell前缀格式化工具

> **一句话总结**：格式化Shell前缀命令，处理可执行路径和参数的组合引用。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/shellPrefix.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~29行 |
| 主要职责 | 格式化Shell前缀命令字符串 |

---

## 功能概述

该模块提供单个实用函数`formatShellPrefixCommand`，用于将Shell前缀（可执行路径+可选参数）与要执行的命令组合成正确引用的完整命令。

处理场景：
- `bash` → `'bash' 'command'`
- `/usr/bin/bash -c` → `'/usr/bin/bash' -c 'command'`
- `C:\Program Files\Git\bin\bash.exe -c` → `'C:\Program Files\Git\bin\bash.exe' -c 'command'`

---

## 核心内容详解

### 导出函数

- **`formatShellPrefixCommand`**
  - 参数：
    - `prefix`（string）- Shell前缀（可执行文件+可选参数）
    - `command`（string）- 要执行的命令
  - 返回值：`string` - 格式化后的完整命令

### 实现逻辑

1. 查找最后一个` -`（空格+连字符）位置
2. 如果存在，分割为：
   - `execPath` = 前缀部分（可执行文件路径）
   - `args` = 后缀部分（参数如-c）
3. 使用`quote()`分别引用可执行路径和命令
4. 参数部分保持原样（已知的有效参数）

---

## 与其他文件的关系

- **依赖**：
  - `./shellQuote.ts` - quote函数
- **被依赖**：
  - `../Shell.ts` - 构建执行命令

---

## 注意事项

- 假设参数部分（-c等）是安全的，不进行额外引用
- 专门处理Windows路径中的空格
- 仅用于格式化，不验证路径是否存在
