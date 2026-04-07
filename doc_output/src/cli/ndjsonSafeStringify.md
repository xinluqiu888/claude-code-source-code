# ndjsonSafeStringify.ts — NDJSON安全序列化工具

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/ndjsonSafeStringify.ts` |
| **文件类型** | TypeScript 工具模块 |
| **行数** | 33 行 |
| **职责** | 提供对 NDJSON（Newline Delimited JSON）流安全的 JSON 序列化功能 |

## 功能概述

本模块解决了一个特定的 JSON 序列化边界问题：JavaScript 的 `JSON.stringify` 会原样输出 Unicode 行分隔符 `U+2028`（LINE SEPARATOR）和段落分隔符 `U+2029`（PARAGRAPH SEPARATOR）。虽然这在 ECMA-404 标准中是合法的，但对于使用 JavaScript 行终止符语义（包括 `\n`、`\r`、`U+2028`、`U+2029`）来分割流的接收方来说，这些字符会导致 JSON 在中间被切断，造成消息丢失。

模块通过将这些 Unicode 字符转义为 `\uXXXX` 形式来解决此问题。这种转义后的形式与原字符等价（解析后得到相同的字符串），但不会被任何接收方误识别为行终止符。这是 ES2019 的 "Subsume JSON" 提案和 Node.js 的 `util.inspect` 采用的方法。

## 核心内容详解

### 导入

| 导入项 | 来源 | 用途 |
|--------|------|------|
| `jsonStringify` | `../utils/slowOperations.js` | 底层 JSON 序列化函数（可能是异步/慢操作包装） |

### 常量

| 名称 | 类型 | 描述 |
|------|------|------|
| `JS_LINE_TERMINATORS` | `RegExp` | 匹配 `U+2028` 或 `U+2029` 的正则表达式，使用单正则配合交替（alternation）以提高性能 |

### 函数

#### `escapeJsLineTerminators(json: string): string`

**功能**：将字符串中的 JavaScript 行终止符转义为 Unicode 转义序列。

**参数**：
- `json`: 要转义的 JSON 字符串

**实现细节**：
- 使用 `String.prototype.replace` 配合正则表达式
- 回调函数根据匹配字符返回 `\u2028` 或 `\u2029`
- 单次正则匹配比两次完整字符串扫描更高效

#### `ndjsonSafeStringify(value: unknown): string`

**功能**：为每行一条消息的传输格式提供安全的 JSON 序列化。

**参数**：
- `value`: 要序列化的任意值

**返回值**：转义后的 JSON 字符串

**执行流程**：
1. 调用 `jsonStringify(value)` 进行基础序列化
2. 调用 `escapeJsLineTerminators()` 转义行终止符
3. 返回处理后的字符串

## 设计要点

1. **性能优化**：使用单个带交替的正则表达式，而非两次独立的扫描，减少性能开销
2. **保持 JSON 有效性**：转义后的输出仍然是合法的 JSON，解析后得到相同的值
3. **通用兼容性**：`\uXXXX` 形式永远不会被任何接收方误识别为行终止符
4. **问题背景**：ProcessTransport 现在会静默跳过非 JSON 行而不是崩溃（gh-28405），但截断的片段仍然会丢失——转义确保消息不会被截断

## 与其他文件的关系

| 关系类型 | 文件 | 描述 |
|---------|------|------|
| **导入** | `src/utils/slowOperations.js` | 使用 `jsonStringify` 进行基础序列化 |
| **被导入** | `src/cli/structuredIO.ts` | 用于 SDK 消息输出序列化 |
| **被导入** | `src/cli/remoteIO.ts` | 用于远程 IO 消息输出序列化 |
| **被导入** | `src/cli/print.ts` | 用于 headless 模式的消息输出 |

## 注意事项

1. **仅处理特定字符**：只处理 `U+2028` 和 `U+2029`，不处理其他控制字符
2. **与标准 JSON.stringify 的区别**：标准实现会原样输出这些字符，而此模块进行显式转义
3. **向后兼容**：转义后的 JSON 仍然可以被所有标准 JSON 解析器正确解析
4. **性能考虑**：正则替换会稍微降低序列化性能，但对于流式传输的可靠性来说是必要的权衡
