# addCommand.ts — MCP添加子命令

> **一句话总结**：实现 `mcp add` CLI子命令，支持stdio、SSE和HTTP传输类型。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/mcp/addCommand.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 281 行 |
| 主要职责 | 注册并处理 `mcp add` 子命令 |

---

## 功能概述

该文件实现了 `claude mcp add` CLI子命令，用于向Claude Code添加MCP服务器。支持三种传输类型：stdio（标准输入输出）、SSE（服务器发送事件）和HTTP。还支持OAuth认证配置和XAA（SEP-990）身份提供者集成。

---

## 核心内容详解

### 导入与依赖

```typescript
import { type Command, Option } from '@commander-js/extra-typings'
import { cliError, cliOk } from '../../cli/exit.js'
import { type AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS, logEvent } from '../../services/analytics/index.js'
import { readClientSecret, saveMcpClientSecret } from '../../services/mcp/auth.js'
import { addMcpConfig } from '../../services/mcp/config.js'
import { describeMcpConfigFilePath, ensureConfigScope, ensureTransport, parseHeaders } from '../../services/mcp/utils.js'
import { getXaaIdpSettings, isXaaEnabled } from '../../services/mcp/xaaIdpLogin.js'
import { parseEnvVars } from '../../utils/envUtils.js'
import { jsonStringify } from '../../utils/slowOperations.js'
```

### 主要函数

#### registerMcpAddCommand(mcp: Command): void

- **类型**: `function`
- **参数**: `mcp` - Commander命令对象
- **用途**: 在给定的MCP命令上注册 `add` 子命令

**命令参数**:

- `<name>`: 服务器名称（必需）
- `<commandOrUrl>`: 命令或URL（必需）
- `[args...]`: 额外参数（可选）

**选项**:

- `-s, --scope <scope>`: 配置范围（local、user、project），默认 `local`
- `-t, --transport <transport>`: 传输类型（stdio、sse、http）
- `-e, --env <env...>`: 环境变量（如 `-e KEY=value`）
- `-H, --header <header...>`: WebSocket请求头
- `--client-id <clientId>`: OAuth客户端ID
- `--client-secret`: 提示输入OAuth客户端密钥
- `--callback-port <port>`: OAuth回调固定端口
- `--xaa`: 启用XAA（SEP-990），需要 `CLAUDE_CODE_ENABLE_XAA=1`

**执行流程**:

1. 验证服务器名称和命令/URL是否存在
2. 验证配置范围和传输类型
3. 如果是XAA模式，验证XAA是否启用以及必需的参数
4. 检查传输类型是否为显式指定
5. 检查命令是否看起来像URL（警告用户）
6. 记录分析事件
7. 根据传输类型处理：
   - SSE/HTTP: 配置OAuth、处理密钥、添加配置
   - stdio: 解析环境变量、添加配置

---

## 设计要点

1. **传输类型推断**: 如果没有显式指定传输类型，会根据输入内容警告用户可能的误用。

2. **XAA支持**: XAA功能需要环境变量 `CLAUDE_CODE_ENABLE_XAA=1` 启用。

3. **URL检测**: 检测输入是否看起来像URL，提示用户使用正确的传输类型。

4. **分析追踪**: 记录服务器添加事件，包含传输类型、范围、是否为显式传输等信息。

---

## 与其他文件的关系

**依赖**:
- `cliError`, `cliOk` (`../../cli/exit.js`)
- `logEvent` (`../../services/analytics/index.js`)
- `readClientSecret`, `saveMcpClientSecret` (`../../services/mcp/auth.js`)
- `addMcpConfig` (`../../services/mcp/config.js`)
- `ensureConfigScope`, `ensureTransport`, `parseHeaders` (`../../services/mcp/utils.js`)
- `getXaaIdpSettings`, `isXaaEnabled` (`../../services/mcp/xaaIdpLogin.js`)
- `parseEnvVars` (`../../utils/envUtils.js`)

**被依赖**:
- CLI命令注册系统

---

## 注意事项

1. **XAA限制**: XAA功能处于实验状态，需要显式启用。

2. **OAuth密钥**: 使用 `--client-secret` 时会从环境变量 `MCP_CLIENT_SECRET` 读取密钥。

3. **URL警告**: 如果输入看起来像URL但没有指定传输类型，会显示警告。

4. **stdio限制**: OAuth相关选项仅支持HTTP/SSE传输类型。
