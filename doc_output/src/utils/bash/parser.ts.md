# parser.ts — Tree-sitter Bash解析器接口

> **一句话总结**：提供Tree-sitter解析器的异步初始化、命令解析和AST遍历功能，支持安全分析和命令提取。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/parser.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~231行 |
| 主要职责 | 封装Tree-sitter NAPI模块，提供命令解析和AST分析接口 |

---

## 功能概述

该模块是Tree-sitter Bash解析器的TypeScript封装层。它提供了：
1. 异步初始化和加载原生NAPI模块
2. 命令字符串解析为AST
3. 从AST提取命令节点和环境变量
4. 原始解析（跳过额外分析）用于性能敏感场景

模块设计考虑了：
- 特性门控：仅在`feature('TREE_SITTER_BASH')`启用时加载
- 错误处理：解析失败时返回null或PARSE_ABORTED符号
- 性能：最大命令长度限制、解析超时保护

---

## 核心内容详解

### 主要导出类型

- **`Node`**
  - 类型：类型别名（TsNode）
  - 用途：Tree-sitter AST节点的TypeScript类型

- **`ParsedCommandData`**
  - 类型：接口
  - 属性：
    - `rootNode`: Node - AST根节点
    - `envVars`: string[] - 环境变量赋值数组
    - `commandNode`: Node | null - 主命令节点
    - `originalCommand`: string - 原始命令字符串

### 主要导出函数

- **`ensureInitialized`**
  - 返回值：`Promise<void>`
  - 用途：确保解析器已初始化（幂等操作）

- **`parseCommand`**
  - 参数：`command`（string）
  - 返回值：`Promise<ParsedCommandData | null>`
  - 用途：完整解析命令，提取环境变量和命令节点
  - 特性：门控在TREE_SITTER_BASH特性后

- **`parseCommandRaw`**
  - 参数：`command`（string）
  - 返回值：`Promise<Node | null | typeof PARSE_ABORTED>`
  - 用途：原始解析，仅返回AST根节点
  - 特性：性能更优，用于安全验证路径
  - 特殊返回值：`PARSE_ABORTED`表示解析被中止（超时/节点限制）

- **`extractCommandArguments`**
  - 参数：`commandNode`（Node）
  - 返回值：`string[]`
  - 用途：从命令节点提取参数列表
  - 处理：支持声明命令（export/declare等）和普通命令

### 内部函数

- **`findCommandNode`**
  - 用途：在AST中查找主命令节点
  - 处理：变量赋值、管道、重定向语句等复杂情况

- **`extractEnvVars`**
  - 用途：从命令节点提取环境变量赋值

- **`stripQuotes`**
  - 用途：去除字符串两端的引号

---

## 设计要点

1. **特性门控**：所有功能都门控在`feature('TREE_SITTER_BASH')`后
2. **安全符号**：`PARSE_ABORTED`符号区分"未加载"和"解析失败"
3. **性能保护**：最大命令长度（10000）、解析超时（50ms）、节点限制（50000）
4. **延迟加载**：原生模块仅在首次解析时加载

---

## 与其他文件的关系

- **依赖**：
  - `./bashParser.ts` - 原生NAPI模块接口
  - `bun:bundle`的`feature`函数 - 特性门控
- **被依赖**：
  - `./ParsedCommand.ts` - 主要调用者
  - `./prefix.ts` - 命令前缀提取
  - `ast.ts` - 安全验证

---

## 注意事项

- `PARSE_ABORTED`必须被视为"失败关闭"（fail-closed），不能回退到正则路径
- 解析超时可能由恶意输入触发（如`(( a[0][0]... ))`深度嵌套）
- 仅在ant构建中可用，外部构建将回退到正则路径
