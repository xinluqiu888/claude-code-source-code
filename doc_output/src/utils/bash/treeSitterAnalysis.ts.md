# treeSitterAnalysis.ts — Tree-sitter AST安全分析器

> **一句话总结**：从Tree-sitter AST提取安全相关信息（引号上下文、复合结构、危险模式），用于命令安全验证。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/treeSitterAnalysis.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~507行 |
| 主要职责 | Tree-sitter AST安全分析和信息提取 |

---

## 功能概述

该模块是Claude Code命令安全验证的核心分析引擎。它从Tree-sitter AST提取：
1. **引号上下文**：识别单引号、双引号、heredoc的位置
2. **复合结构**：检测管道、子shell、命令组、复合操作符
3. **危险模式**：识别命令替换`$()`、进程替换`<()`、参数扩展`${}`等

相比正则表达式，Tree-sitter提供精确的语法树，避免了引号内的误匹配。

---

## 核心内容详解

### 主要导出类型

- **`QuoteContext`** - 引号上下文
  - `withDoubleQuotes`: 移除单引号内容，保留双引号
  - `fullyUnquoted`: 移除所有引号内容
  - `unquotedKeepQuoteChars`: 保留引号字符，移除内容

- **`CompoundStructure`** - 复合命令结构
  - `hasCompoundOperators`: 含&&/||/;
  - `hasPipeline`: 含管道
  - `hasSubshell`: 含子shell
  - `hasCommandGroup`: 含命令组
  - `operators`: 操作符列表
  - `segments`: 分割后的命令段

- **`DangerousPatterns`** - 危险模式
  - `hasCommandSubstitution`: $()或``
  - `hasProcessSubstitution`: <()或>()
  - `hasParameterExpansion`: ${}
  - `hasHeredoc`: <<EOF
  - `hasComment`: #注释

- **`TreeSitterAnalysis`** - 完整分析结果
  - 包含上述所有类型

### 主要导出函数

- **`extractQuoteContext`**
  - 用途：提取引号上下文
  - 技术：单次遍历收集所有引号span

- **`extractCompoundStructure`**
  - 用途：提取复合命令结构
  - 处理：list节点、pipeline、subshell等

- **`hasActualOperatorNodes`**
  - 用途：检查AST中是否真实存在操作符节点
  - 关键：区分`\;`（转义）和`;`（真实操作符）

- **`extractDangerousPatterns`**
  - 用途：提取危险模式信息

- **`analyzeCommand`**
  - 用途：一次性执行所有分析
  - 返回：`TreeSitterAnalysis`

---

## 与其他文件的关系

- **依赖**：
  - 无（纯AST处理）
- **被依赖**：
  - `ast.ts` - 安全验证主入口
  - `./ParsedCommand.ts` - 解析命令获取AST

---

## 注意事项

- 所有函数接受unknown类型的rootNode（来自原生模块）
- 遍历时注意避免重复计数嵌套结构
- Span处理需要考虑UTF-8字节偏移
- 是安全验证的关键输入，结果影响是否放行命令
