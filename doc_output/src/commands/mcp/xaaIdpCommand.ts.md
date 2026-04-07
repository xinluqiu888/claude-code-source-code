# xaaIdpCommand.ts — XAA IdP连接管理

> **一句话总结**：实现 `claude mcp xaa` 子命令，管理SEP-990 XAA身份提供者连接。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/mcp/xaaIdpCommand.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 267 行 |
| 主要职责 | 管理XAA IdP连接的配置、登录和状态查看 |

---

## 功能概述

该文件实现了 `claude mcp xaa` CLI子命令，用于管理XAA（SEP-990）身份提供者连接。IdP连接是用户级别的，配置一次后，所有启用XAA的MCP服务器都可以复用。配置存储在 `settings.xaaIdp`（非机密）和按issuer分区的keychain槽位（机密）中。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '@commander-js/extra-typings'
import { cliError, cliOk } from '../../cli/exit.js'
import { acquireIdpIdToken, clearIdpClientSecret, clearIdpIdToken, getCachedIdpIdToken, getIdpClientSecret, getXaaIdpSettings, issuerKey, saveIdpClientSecret, saveIdpIdTokenFromJwt } from '../../services/mcp/xaaIdpLogin.js'
import { errorMessage } from '../../utils/errors.js'
import { updateSettingsForSource } from '../../utils/settings/settings.js'
```

### 主要函数

#### registerMcpXaaIdpCommand(mcp: Command): void

- **类型**: `function`
- **参数**: `mcp` - Commander命令对象
- **用途**: 注册XAA IdP相关的子命令

**子命令**:

1. **setup**: 配置IdP连接（一次性设置）
   - `--issuer <url>`: IdP issuer URL（OIDC发现，必需）
   - `--client-id <id>`: Claude Code的client_id（必需）
   - `--client-secret`: 从环境变量读取客户端密钥
   - `--callback-port <port>`: 固定回环回调端口

2. **login**: 缓存IdP id_token使XAA服务器静默认证
   - `--force`: 忽略缓存的id_token重新登录
   - `--id-token <jwt>`: 直接写入预获取的JWT（用于测试）

3. **show**: 显示当前IdP连接配置

4. **clear**: 清除IdP连接配置和缓存的id_token

---

## 设计要点

1. **验证优先**: 在写入前验证所有输入，防止配置写入一半失败导致不一致状态。

2. **URL协议限制**: 只允许HTTPS协议，回环地址可使用HTTP（用于测试）。

3. **callbackPort验证**: 必须是正整数，符合Zod的 `.positive()` 验证。

4. **旧配置清理**: 设置新issuer时清除旧issuer的keychain槽位。

5. **issuerKey归一化**: 使用 `issuerKey()` 处理issuer URL的尾部斜杠和大小写差异。

6. **clientId变更处理**: 同一issuer但不同clientId时，清除旧的id_token和密钥。

---

## 与其他文件的关系

**依赖**:
- `cliError`, `cliOk` (`../../cli/exit.js`)
- `acquireIdpIdToken`, `getXaaIdpSettings`, `saveIdpClientSecret` 等 (`../../services/mcp/xaaIdpLogin.js`)
- `errorMessage` (`../../utils/errors.js`)
- `updateSettingsForSource` (`../../utils/settings/settings.js`)

**被依赖**:
- MCP CLI命令注册

---

## 注意事项

1. **分离信任域**: IdP密钥与每个服务器的AS密钥是分离的信任域。

2. **环境变量**: `--client-secret` 从 `MCP_XAA_IDP_CLIENT_SECRET` 环境变量读取。

3. **HTTP限制**: 仅允许回环地址（localhost、127.0.0.1、[::1]）使用HTTP，其他必须使用HTTPS。

4. **设置写入**: 使用 `updateSettingsForSource` 进行深度合并，需要显式设置 `undefined` 来删除键。

5. **测试支持**: `--id-token` 选项用于一致性测试/e2e测试，mock IdP不服务 `/authorize` 的情况。
