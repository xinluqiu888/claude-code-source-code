# ParsedCommand.ts — Shell命令解析器

> **一句话总结**：提供Shell命令的解析功能，支持Tree-sitter和正则表达式两种解析方式，用于安全分析和命令处理。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/ParsedCommand.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~319行 |
| 主要职责 | 解析Shell命令，提取管道段、重定向信息，支持安全验证 |

---

## 功能概述

ParsedCommand模块是Claude Code中用于处理Shell命令的核心解析器。它提供了统一的接口来处理命令字符串，支持两种底层实现：

1. **Tree-sitter解析**（主要）：使用原生NAPI模块进行精确的语法树解析
2. **正则表达式回退**（遗留）：当Tree-sitter不可用时的备用方案

该模块主要用于：
- 提取命令中的管道段（pipe segments）
- 识别输出重定向（> 和 >>）
- 获取Tree-sitter语法分析数据用于安全验证
- 支持复杂命令结构的解析（如包含引号、变量等的命令）

---

## 核心内容详解

### 主要接口

- **`IParsedCommand`**
  - 类型：接口
  - 用途：定义解析命令的统一接口
  - 方法：`toString()`, `getPipeSegments()`, `withoutOutputRedirections()`, `getOutputRedirections()`, `getTreeSitterAnalysis()`

- **`ParsedCommand`**
  - 类型：对象（命名空间模式）
  - 用途：主入口点，提供`parse()`方法解析命令
  - 特性：具有单条目缓存，避免重复解析相同命令

### 实现类

- **`TreeSitterParsedCommand`**
  - 类型：类
  - 用途：基于Tree-sitter AST的解析实现
  - 关键特性：正确处理UTF-8/UTF-16编码差异，使用Buffer进行字节级切片

- **`RegexParsedCommand_DEPRECATED`**
  - 类型：类
  - 用途：基于正则表达式和shell-quote的回退实现
  - 状态：已弃用，仅当Tree-sitter不可用时使用

### 核心函数

- **`buildParsedCommandFromRoot`**
  - 参数：`command`（字符串）, `root`（Node对象）
  - 返回值：`IParsedCommand`
  - 用途：从预解析的AST根节点构建ParsedCommand实例

- **`getTreeSitterAvailable`**
  - 返回值：`Promise<boolean>`
  - 用途：检测Tree-sitter解析器是否可用
  - 特性：结果被memoize缓存

---

## 设计要点

1. **双轨架构**：Tree-sitter路径提供精确解析，正则路径保证兼容性
2. **缓存机制**：单条目LRU缓存避免重复解析（lastCmd/lastResult）
3. **编码安全**：使用Buffer处理UTF-8字节偏移，避免多字节字符问题
4. **安全性优先**：所有解析都服务于安全验证流程（ast.ts中的parseForSecurity）

---

## 与其他文件的关系

- **依赖**：
  - `./commands.ts` - 命令分割和重定向提取
  - `./parser.ts` - Tree-sitter解析器接口
  - `./treeSitterAnalysis.ts` - 语法树分析
- **被依赖**：
  - `ast.ts` - 安全验证的主入口
  - `bashPipeCommand.ts` - 管道命令处理

---

## 注意事项

- Tree-sitter路径仅在`feature('TREE_SITTER_BASH')`为true时启用
- 正则回退实现可能存在解析差异，仅用于兼容性
- 命令长度限制为10000字符（与parser.ts一致）
