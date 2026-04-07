# shellQuote.ts — Shell-quote安全包装器

> **一句话总结**：为shell-quote库提供安全包装，添加输入验证、错误处理和畸形token检测，防止命令注入攻击。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/shellQuote.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~305行 |
| 主要职责 | 安全地引用和解析Shell命令参数 |

---

## 功能概述

该模块是对shell-quote库的增强包装，添加了多项安全功能：
1. **错误处理**：将异常转换为结果对象而非抛出
2. **类型验证**：验证参数类型，拒绝对象、符号、函数等不安全类型
3. **畸形Token检测**：识别可能导致命令注入的歧义解析
4. **单引号Bug检测**：检测利用shell-quote单引号处理差异的攻击

该模块是Claude Code命令执行安全链的关键环节，所有Shell参数引用都应通过此模块。

---

## 核心内容详解

### 主要导出类型

- **`ShellParseResult`** - 解析结果联合类型
- **`ShellQuoteResult`** - 引用结果联合类型
- **`ParseEntry`** - 来自shell-quote的解析条目类型

### 主要导出函数

- **`tryParseShellCommand`**
  - 参数：`cmd`（string）, `env?`（对象或函数）
  - 返回值：`ShellParseResult`
  - 用途：安全解析Shell命令

- **`tryQuoteShellArgs`**
  - 参数：`args`（unknown[]）
  - 返回值：`ShellQuoteResult`
  - 用途：安全引用Shell参数
  - 验证：拒绝object/symbol/function类型

- **`quote`**
  - 参数：`args`（ReadonlyArray<unknown>）
  - 返回值：`string`
  - 用途：主引用函数，失败时使用lenient fallback

### 安全检测函数

- **`hasMalformedTokens`**
  - 用途：检测解析后的token是否畸形（unbalanced braces/quotes）
  - 安全背景：防止HackerOne #3482049类注入攻击
  - 检测：未终止引号、不平衡括号等

- **`hasShellQuoteSingleQuoteBug`**
  - 用途：检测单引号+反斜杠的差异利用
  - 背景：shell-quote在单引号内错误处理反斜杠
  - 模式：`'\' payload '\'`会被合并为单个token

---

## 与其他文件的关系

- **依赖**：
  - `shell-quote`库
  - `../log.ts` - 错误日志
  - `../slowOperations.ts` - jsonStringify
- **被依赖**：
  - `./bashPipeCommand.ts` - 管道命令处理
  - `./shellQuoting.ts` - 高级引用功能
  - `../Shell.ts` - 所有Shell执行

---

## 注意事项

- hasMalformedTokens是关键安全检查，不能跳过
- 单引号Bug检测需要遍历字符状态机
- quote函数在失败时会尝试lenient fallback
- JSON.stringify不应用于Shell引用（会产生双引号字符串）
