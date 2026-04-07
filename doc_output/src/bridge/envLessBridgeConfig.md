# envLessBridgeConfig.ts — 无环境桥接配置管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/bridge/envLessBridgeConfig.ts`
- **文件类型**: TypeScript 模块
- **行数**: 约 166 行
- **主要职责**: 管理无环境（v2）桥接的远程配置，包括重试策略、超时设置和版本检查

## 功能概述

`envLessBridgeConfig.ts` 负责管理无环境（env-less）桥接模式的远程配置。该模块从 GrowthBook 功能标志系统获取配置，支持动态调整桥接行为而无需代码部署。这是 v2 桥接架构的关键组件，与基于环境的 v1 桥接配置分离。

配置涵盖多个方面：初始化阶段的重试策略（创建会话、POST /bridge、恢复）；HTTP 超时设置；UUID 去重缓冲区大小；工作器心跳节奏；JWT 刷新缓冲；归档超时；连接超时；以及最低版本要求。

模块使用 Zod 进行严格的运行时验证，确保从 GrowthBook 获取的配置值在有效范围内。无效配置会回退到默认值，保证系统稳定性。

## 核心内容详解

### EnvLessBridgeConfig 类型

完整的配置类型定义：

```typescript
type EnvLessBridgeConfig = {
  // 初始化阶段重试配置
  init_retry_max_attempts: number        // 最大重试次数
  init_retry_base_delay_ms: number       // 基础延迟
  init_retry_jitter_fraction: number     // 抖动比例
  init_retry_max_delay_ms: number        // 最大延迟
  
  // HTTP 超时
  http_timeout_ms: number                // 通用 HTTP 超时
  
  // UUID 去重
  uuid_dedup_buffer_size: number         // 去重缓冲区大小
  
  // 心跳配置
  heartbeat_interval_ms: number          // 心跳间隔（服务器 TTL 60s）
  heartbeat_jitter_fraction: number      // 心跳抖动比例
  
  // JWT 刷新
  token_refresh_buffer_ms: number        // 过期前刷新缓冲
  
  // 关闭阶段
  teardown_archive_timeout_ms: number    // 归档超时（独立设置）
  
  // 连接超时
  connect_timeout_ms: number             // 连接超时（用于遥测）
  
  // 版本要求
  min_version: string                    // 最低 CLI 版本
  should_show_app_upgrade_message: boolean // 是否显示应用升级提示
}
```

### 默认值常量

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `init_retry_max_attempts` | 3 | 初始化重试次数 |
| `init_retry_base_delay_ms` | 500 | 基础退避延迟 |
| `init_retry_jitter_fraction` | 0.25 | ±25% 抖动 |
| `init_retry_max_delay_ms` | 4000 | 最大退避延迟 |
| `http_timeout_ms` | 10_000 | HTTP 请求超时 |
| `uuid_dedup_buffer_size` | 2000 | 去重环大小 |
| `heartbeat_interval_ms` | 20_000 | 心跳间隔（60s TTL 的 1/3）
| `heartbeat_jitter_fraction` | 0.1 | ±10% 心跳抖动 |
| `token_refresh_buffer_ms` | 300_000 | JWT 刷新缓冲（5分钟）
| `teardown_archive_timeout_ms` | 1500 | 归档超时（1.5秒）
| `connect_timeout_ms` | 15_000 | 连接超时 |
| `min_version` | '0.0.0' | 最低版本（无限制）
| `should_show_app_upgrade_message` | false | 不显示升级提示 |

### Zod 验证模式

严格的运行时验证模式，包含范围限制：

**初始化重试**:
- `init_retry_max_attempts`: 整数，1-10，默认 3
- `init_retry_base_delay_ms`: 整数，≥100，默认 500
- `init_retry_jitter_fraction`: 0-1，默认 0.25
- `init_retry_max_delay_ms`: 整数，≥500，默认 4000

**HTTP 和去重**:
- `http_timeout_ms`: 整数，≥2000，默认 10_000
- `uuid_dedup_buffer_size`: 整数，100-50_000，默认 2000

**心跳**:
- `heartbeat_interval_ms`: 整数，5000-30_000，默认 20_000
  - 服务器 TTL 是 60s，5s 下限防止抖动，30s 上限保持 ≥2x 余量
- `heartbeat_jitter_fraction`: 0-0.5，默认 0.1
  - 上限 0.5：最坏情况 30s × 1.5 = 45s，仍低于 60s TTL

**JWT 刷新**:
- `token_refresh_buffer_ms`: 整数，30_000-1_800_000，默认 300_000
  - 下限 30s 防止紧循环
  - 上限 30分钟防止语义倒置（将"到期前 5 分钟"误配为"延迟 5 分钟"）

**关闭**:
- `teardown_archive_timeout_ms`: 整数，500-2000，默认 1500
  - 上限 2000 保持在 gracefulShutdown 的 2s 清理竞速内

**连接**:
- `connect_timeout_ms`: 整数，5000-60_000，默认 15_000
  - 观察到的 p99 连接时间是 2-3s，15s 提供 ~5x 余量

**版本**:
- `min_version`: 字符串，必须是有效 semver，默认 '0.0.0'
- `should_show_app_upgrade_message`: 布尔值，默认 false

### getEnvLessBridgeConfig 函数

**签名**: `getEnvLessBridgeConfig(): Promise<EnvLessBridgeConfig>`

**功能**: 从 GrowthBook 获取无环境桥接配置。

**行为**:
1. 调用 `getFeatureValue_DEPRECATED` 获取 `tengu_bridge_repl_v2_config`
2. 使用默认值作为回退
3. 使用 Zod 模式验证返回数据
4. 验证失败则回退到默认配置

**注意**: 使用 `_DEPRECATED` 后缀的阻塞获取器，因为 `/remote-control` 在 GrowthBook 初始化后运行，可以立即获得内存中的远程评估值。

### checkEnvLessBridgeMinVersion 函数

**签名**: `checkEnvLessBridgeMinVersion(): Promise<string | null>`

**功能**: 检查当前 CLI 版本是否满足 v2 桥接的最低版本要求。

**返回**:
- 如果版本过低：错误消息字符串（包含升级建议）
- 如果版本正常：`null`

**比较**: 使用 `lt`（小于）semver 比较，与 `checkBridgeMinVersion()` 分离，使 v1 和 v2 可以独立强制升级。

### shouldShowAppUpgradeMessage 函数

**签名**: `shouldShowAppUpgradeMessage(): Promise<boolean>`

**功能**: 判断是否应该提示用户升级 claude.ai 应用。

**用途**: 当 v2 桥接活跃且配置位设置时，提示用户应用可能太旧而无法看到 v2 会话列表。这允许在应用支持新会话列表查询之前推出 v2 桥接。

**条件**:
1. `isEnvLessBridgeEnabled()` 返回 true
2. `config.should_show_app_upgrade_message` 为 true

## 设计要点

1. **防御性验证**: Zod 模式拒绝整个无效对象而非部分信任，与 `pollConfig.ts` 采用相同的防御深度策略。

2. **范围限制**: 每个数值配置都有明确的上下限，防止配置错误导致系统不稳定。

3. **语义保护**: `token_refresh_buffer_ms` 的上限 30 分钟专门用于捕获"到期前 5 分钟"和"延迟 5 分钟"的语义混淆。

4. **竞速意识**: `teardown_archive_timeout_ms` 上限 2s 确保在 gracefulShutdown 的强制退出前完成。

5. **分离版本线**: v2 使用独立的 `tengu_bridge_repl_v2_config` 标志，与 v1 的 `tengu_bridge_min_version` 分离，支持独立升级。

6. **心跳数学**: 20s 心跳间隔在 60s TTL 下提供 3x 余量，加上 ±10% 抖动，最坏情况 22s 仍低于 TTL。

7. **使用阻塞 API**: 使用 `getFeatureValue_DEPRECATED` 而非缓存版本，确保获得最新的远程评估值。

## 与其他文件的关系

| 文件 | 关系描述 |
|------|----------|
| `../services/analytics/growthbook.js` | 导入 `getFeatureValue_DEPRECATED` |
| `../utils/lazySchema.js` | 导入延迟加载的 Zod 模式 |
| `../utils/semver.js` | 导入 `lt` 进行版本比较 |
| `./bridgeEnabled.js` | 导入 `isEnvLessBridgeEnabled` |
| `replBridge.ts` | 调用 `getEnvLessBridgeConfig` 获取配置 |

## 注意事项

1. **同步配置**: 配置在 `initEnvLessBridgeCore` 调用时读取一次，会话生命周期内固定。动态配置变更需要重新启动桥接。

2. **Zod 默认行为**: Zod 的 `.default()` 在验证失败时替换整个值，而非保留部分有效字段。

3. **版本比较方向**: `lt(MACRO.VERSION, cfg.min_version)` 检查当前版本是否低于要求，注意版本字符串的方向。

4. **GrowthBook 延迟**: 虽然使用阻塞获取器，但首次调用仍可能有 GrowthBook 初始化的延迟。

5. **心跳抖动计算**: 实际抖动范围是 `interval ± (interval * jitter_fraction)`，不是绝对毫秒值。

6. **JWT 刷新时机**: 刷新在 `expires_in - token_refresh_buffer_ms` 时触发，不是基于固定间隔。

7. **归档超时独立性**: `teardown_archive_timeout_ms` 与 `http_timeout_ms` 分离，因为关闭阶段有独立的竞速条件。

8. **遥测价值**: `connect_timeout_ms` 主要价值是遥测——识别那些发出 `started` 事件后无声消失的会话。
