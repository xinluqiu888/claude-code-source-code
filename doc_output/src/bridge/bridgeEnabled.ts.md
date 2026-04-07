# bridgeEnabled.ts — 桥接功能开关控制

> **一句话总结**：通过GrowthBook特性开关控制Remote Control桥接功能的可用性。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/bridge/bridgeEnabled.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 203 |
| 主要职责 | 运行时检查和控制Remote Control桥接功能的启用状态 |

---

## 功能概述

该文件实现了Remote Control（远程控制）功能的权限检查和开关控制。它基于GrowthBook特性开关系统，结合用户订阅状态和OAuth认证信息，决定是否启用桥接功能。

Remote Control需要claude.ai订阅（桥接使用claude.ai OAuth令牌向CCR认证），该模块排除了Bedrock/Vertex/Foundry、API Key部署以及Console API登录等场景。

---

## 核心内容详解

### 导入与依赖

| 依赖 | 来源 | 用途 |
|------|------|------|
| `feature` | `bun:bundle` | 编译时特性标志 |
| `checkGate_CACHED_OR_BLOCKING`, `getDynamicConfig_CACHED_MAY_BE_STALE`, `getFeatureValue_CACHED_MAY_BE_STALE` | `../services/analytics/growthbook.js` | GrowthBook特性开关 |
| `authModule` | `../utils/auth.js` | 认证工具（命名空间导入避免循环依赖） |
| `isEnvTruthy` | `../utils/envUtils.js` | 环境变量解析 |
| `lt` | `../utils/semver.js` | 语义化版本比较 |

### 主要函数

#### `isBridgeEnabled`
- **类型**: `function`
- **用途**: 非阻塞检查桥接功能是否启用（快速路径）
- **返回值**: `boolean`
- **条件**: 
  - 编译时 `BRIDGE_MODE` 特性开启
  - 用户是claude.ai订阅者
  - GrowthBook `tengu_ccr_bridge` 特性为true
- **关键逻辑**: 使用三目运算符的"正模式"（见docs/feature-gating.md）

#### `isBridgeEnabledBlocking`
- **类型**: `async function`
- **用途**: 阻塞式权限检查（慢速路径）
- **返回值**: `Promise<boolean>`
- **行为**: 
  - 缓存为true时立即返回
  - 缓存为false或缺失时，等待GrowthBook初始化（最多~5秒）
  - 获取服务器最新值并写入磁盘缓存

#### `getBridgeDisabledReason`
- **类型**: `async function`
- **用途**: 获取桥接不可用的具体原因，用于向用户显示可操作的错误信息
- **返回值**: `Promise<string | null>` - 原因描述或null（表示可用）
- **诊断流程**:
  1. 检查是否为claude.ai订阅者
  2. 检查是否有profile scope（长期令牌缺少此scope）
  3. 检查组织UUID是否存在
  4. 检查GrowthBook gate是否开启

#### `isEnvLessBridgeEnabled`
- **类型**: `function`
- **用途**: 检查v2（无环境变量）REPL桥接路径是否启用
- **返回值**: `boolean`
- **控制**: GrowthBook `tengu_bridge_repl_v2` 特性

#### `isCseShimEnabled`
- **类型**: `function`
- **用途**: 控制 `cse_*` → `session_*` 客户端重标记shim的开关
- **返回值**: `boolean`（默认true）
- **背景**: 兼容性shim，等待服务器和前端支持后直接路由 `cse_*`

#### `checkBridgeMinVersion`
- **类型**: `function`
- **用途**: 检查CLI版本是否满足v1桥接的最低要求
- **返回值**: `string | null` - 错误消息或null（版本符合）
- **逻辑**: 比较 `MACRO.VERSION` 与GrowthBook配置的 `minVersion`

#### `getCcrAutoConnectDefault`
- **类型**: `function`
- **用途**: 获取CCR自动连接的默认设置
- **返回值**: `boolean`
- **条件**: 编译时 `CCR_AUTO_CONNECT` 特性 + GrowthBook `tengu_cobalt_harbor` gate

#### `isCcrMirrorEnabled`
- **类型**: `function`
- **用途**: 检查CCR镜像模式是否启用
- **返回值**: `boolean`
- **控制**: 环境变量 `CLAUDE_CODE_CCR_MIRROR` 或GrowthBook `tengu_ccr_mirror`

### 内部辅助函数

| 函数 | 用途 | 异常处理 |
|------|------|---------|
| `isClaudeAISubscriber` | 检查是否为claude.ai订阅者 | 捕获异常返回false（预配置阶段） |
| `hasProfileScope` | 检查OAuth令牌是否有profile scope | 捕获异常返回false |
| `getOauthAccountInfo` | 获取OAuth账户信息 | 捕获异常返回undefined |

---

## 设计要点

1. **命名空间导入**: 使用 `import * as authModule` 而非直接导入，打破 `bridgeEnabled → auth → config → bridgeEnabled` 循环依赖
2. **正模式特性门控**: 使用三元运算符的正模式确保外部构建中消除字符串字面量
3. **多级缓存策略**: 区分 `CACHED_MAY_BE_STALE`（快速但可能过期）和 `CACHED_OR_BLOCKING`（准确但可能等待）
4. **防御性编程**: 预配置阶段捕获异常返回安全默认值

---

## 与其他文件的关系

- **依赖**:
  - `services/analytics/growthbook.js` - 特性开关系统
  - `utils/auth.js` - 认证检查（延迟绑定）
  - `utils/envUtils.js` - 环境变量工具
  - `utils/semver.js` - 版本比较
- **被依赖**: 由主入口、桥接初始化、命令处理等调用

---

## 注意事项

1. **OAuth Scope要求**: Remote Control需要 `user:profile` scope，setup-token和CLAUDE_CODE_OAUTH_TOKEN环境变量缺少此scope
2. **版本兼容性**: v1和v2桥接实现有独立的最低版本要求
3. **显式设置优先**: `remoteControlAtStartup` 显式设置始终覆盖 `getCcrAutoConnectDefault` 的默认值
4. **组织UUID依赖**: GrowthBook gate基于organizationUUID，来自 `/api/oauth/profile`
