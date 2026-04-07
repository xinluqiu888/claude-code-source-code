# exit.ts — CLI退出辅助工具

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/exit.ts` |
| **文件类型** | TypeScript 工具模块 |
| **行数** | 32 行 |
| **职责** | 为 CLI 子命令处理器提供统一的退出辅助函数 |

## 功能概述

本模块是 CLI 工具集中的退出辅助模块，旨在简化和统一 `claude mcp *` 和 `claude plugin *` 等子命令处理器的退出流程。它解决了之前在处理 CLI 退出时反复出现的 4-5 行代码复制粘贴问题，将退出逻辑集中管理。

模块提供了两个核心函数 `cliError` 和 `cliOk`，分别用于错误退出和正常退出。通过 `never` 返回类型，TypeScript 可以在调用点进行控制流窄化，无需在调用后添加 `return` 语句。

设计考虑了测试友好性：函数使用 `console.error` 和 `process.stdout.write` 进行输出，允许测试代码通过间谍（spy）方式进行验证，而不需要实际退出进程。

## 核心内容详解

### 导入

- 无外部导入，纯原生 Node.js 实现。

### 常量与配置

| 名称 | 类型 | 描述 |
|------|------|------|
| `eslint-disable custom-rules/no-process-exit` | 注释指令 | 禁用自定义规则，因为这是集中式 CLI 退出点 |

### 函数

#### `cliError(msg?: string): never`

**功能**：输出错误信息到 stderr 并以退出码 1 终止进程。

**参数**：
- `msg` (可选): 要输出的错误消息字符串

**行为**：
1. 如果提供了消息，使用 `console.error` 输出到 stderr
2. 调用 `process.exit(1)` 终止进程
3. 返回 `undefined as never` 以满足 TypeScript 类型系统

**设计要点**：
- 使用 `console.error` 便于测试中通过间谍监听
- `: never` 返回类型确保调用后的代码被 TypeScript 识别为不可达

#### `cliOk(msg?: string): never`

**功能**：输出消息到 stdout 并以退出码 0 正常终止进程。

**参数**：
- `msg` (可选): 要输出的消息字符串

**行为**：
1. 如果提供了消息，使用 `process.stdout.write` 输出到 stdout（带换行）
2. 调用 `process.exit(0)` 终止进程
3. 返回 `undefined as never` 以满足 TypeScript 类型系统

**设计要点**：
- 使用 `process.stdout.write` 而非 `console.log`，因为 Bun 的 `console.log` 不经过 `process.stdout.write`
- 便于测试中对 `process.stdout.write` 进行间谍操作

## 设计要点

1. **集中式退出管理**：将分散在约 60 个地方的退出逻辑统一，减少代码重复
2. **类型安全**：`: never` 返回类型允许 TypeScript 进行控制流分析，调用后的代码被视为不可达
3. **测试友好**：使用可间谍化的输出方法（`console.error`、`process.stdout.write`），便于单元测试
4. **无实际返回**：虽然函数体在 `process.exit()` 后有返回语句，但只是为了满足 TypeScript 的类型系统，实际永远不会执行

## 与其他文件的关系

| 关系类型 | 文件 | 描述 |
|---------|------|------|
| **被导入** | `src/cli/handlers/mcp.tsx` | MCP 子命令处理器使用 `cliError` 和 `cliOk` 进行退出 |
| **被导入** | `src/cli/handlers/auth.ts` | 认证处理器使用这些函数 |
| **被导入** | 其他子命令处理器 | 所有 CLI 子命令处理器都可以使用此模块 |

## 注意事项

1. **禁用 ESLint 规则**：文件顶部明确禁用了 `custom-rules/no-process-exit` 规则，因为这是唯一允许直接调用 `process.exit()` 的地方
2. **测试中的特殊处理**：测试中会对 `process.exit` 进行间谍操作，让它返回而不实际退出，因此代码在 `process.exit()` 后还有返回语句
3. **输出方式选择**：`cliError` 使用 `console.error`，`cliOk` 使用 `process.stdout.write`，这种差异是为了适应测试框架的间谍能力
4. **不要在此后添加代码**：由于函数返回 `never`，TypeScript 会将对这些函数的调用视为终止点，调用后的任何代码都将被标记为不可达
