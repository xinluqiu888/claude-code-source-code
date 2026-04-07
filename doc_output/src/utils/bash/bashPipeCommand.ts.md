# bashPipeCommand.ts — 管道命令重排工具

> **一句话总结**：重新排列包含管道的命令，将stdin重定向放置在第一个命令之后，解决eval处理管道命令时的输入重定向问题。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/bashPipeCommand.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~295行 |
| 主要职责 | 修复管道命令中的stdin重定向位置问题 |

---

## 功能概述

该模块解决了一个特定的Shell执行问题：当使用`eval`执行包含管道的命令时，如果stdin重定向位于命令末尾，它会应用到整个管道而非第一个命令。这是因为eval将整个管道视为一个执行单元。

例如：
- 问题形式：`eval 'cmd1 | cmd2' < /dev/null` → 重定向应用到eval本身
- 修复形式：`eval 'cmd1 < /dev/null | cmd2'` → 重定向应用到cmd1

该模块使用shell-quote库解析命令，识别第一个管道操作符，并将stdin重定向插入到正确位置。

---

## 核心内容详解

### 主要导出函数

- **`rearrangePipeCommand`**
  - 参数：`command`（string）- 要重排的命令
  - 返回值：`string` - 重排后的命令
  - 用途：主入口函数，将stdin重定向移到第一个管道命令后

### 辅助函数

- **`findFirstPipeOperator`**
  - 参数：`parsed`（ParseEntry[]）- shell-quote解析结果
  - 返回值：`number` - 第一个管道操作符的索引

- **`buildCommandParts`**
  - 参数：`parsed`, `start`, `end`
  - 用途：从解析结果构建命令片段
  - 特殊处理：正确处理文件描述符重定向（2>&1, 2>/dev/null）

- **`quoteWithEvalStdinRedirect`**
  - 参数：`command`（string）
  - 用途：当无法安全解析时使用eval回退方案

- **`singleQuoteForEval`**
  - 参数：`s`（string）
  - 用途：使用单引号包裹命令，正确处理嵌套单引号

- **`joinContinuationLines`**
  - 参数：`command`（string）
  - 用途：处理反斜杠续行（\\newline）

### 跳过条件

以下情况会直接使用回退方案（不尝试解析重排）：
- 包含反引号（命令替换）
- 包含`$(`（命令替换）
- 包含shell变量（$VAR, ${VAR}）
- 包含bash控制结构（for/while/until/if/case/select）
- 包含换行符（多行命令）
- 存在单引号bug模式（security相关）
- 解析失败或包含畸形token

---

## 设计要点

1. **安全第一**：在不确定的情况下使用保守的回退方案
2. **精确解析**：使用shell-quote库进行token级解析，而非简单的字符串替换
3. **兼容处理**：支持环境变量赋值前缀（VAR=value cmd）
4. **特殊字符处理**：正确处理文件描述符重定向和glob模式

---

## 与其他文件的关系

- **依赖**：
  - `./shellQuote.ts` - shell-quote库的包装和扩展
- **被依赖**：
  - `../Shell.ts` - 在执行命令前调用重排
  - `ast.ts` - 安全验证流程中使用

---

## 注意事项

- 该模块涉及安全关键代码（#9732, #32515等issue）
- 对于包含单引号bug的命令会回退到安全方案
- 无法处理控制结构和复杂变量扩展
- 续行处理使用特定的奇偶检测逻辑
