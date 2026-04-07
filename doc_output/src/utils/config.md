# config.ts — 配置管理系统

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/config.ts`
- **主要功能**: 全局配置和项目配置的读取、保存、管理
- **关键依赖**: `bun:bundle`, `lodash-es`, `lockfile.js`, `envUtils.ts`, `errors.ts`, `slowOperations.ts`

## 功能概述

该模块负责 Claude Code 的全局配置管理，包括：
1. 全局配置（`~/.claude.json`）的读取和写入
2. 项目级配置的管理
3. 配置缓存和新鲜度监控
4. 配置迁移和字段升级
5. 文件锁机制确保并发安全
6. 防数据丢失机制（auth状态保护）

## 核心内容详解

### 配置类型定义

```typescript
// 项目配置类型
export type ProjectConfig = {
  allowedTools: string[]
  mcpContextUris: string[]
  mcpServers?: Record<string, McpServerConfig>
  // ... 更多字段
}

// 全局配置类型
export type GlobalConfig = {
  numStartups: number
  installMethod?: InstallMethod
  theme: ThemeSetting
  verbose: boolean
  oauthAccount?: AccountInfo
  // ... 100+ 个配置字段
}
```

### 核心函数

#### `getGlobalConfig(): GlobalConfig`
- 获取全局配置（带缓存）
- 首次调用从文件读取，后续从缓存返回
- 自动启动新鲜度监控

#### `saveGlobalConfig(updater): void`
- 保存全局配置更新
- 使用文件锁防止并发写入冲突
- 包含防数据丢失检查（防止auth状态被覆盖）

#### `getConfig(file, createDefault): T`
- 通用配置读取函数
- 支持JSON解析错误回退到默认值

#### `checkHasTrustDialogAccepted(): boolean`
- 检查用户是否已接受信任对话框
- 支持向上遍历父目录检查

#### `isPathTrusted(dir): boolean`
- 检查指定路径是否受信任
- 用于权限控制

### 配置缓存机制

```typescript
let globalConfigCache: { config: GlobalConfig | null; mtime: number }
```

- 内存缓存避免重复读取
- `writeThroughGlobalConfigCache`: 写入后立即更新缓存
- `startGlobalConfigFreshnessWatcher`: 监控文件变化（1秒轮询）

### 防数据丢失机制

`wouldLoseAuthState(fresh)` 函数检查：
- 缓存中有 OAuth 账户信息但新配置中没有
- 缓存中已完成 onboarding 但新配置中没有
- 防止并发写入导致配置损坏（GitHub issue #3117）

## 设计要点

1. **并发安全**: 使用 `lockfile.js` 实现文件级锁
2. **缓存优化**: 内存缓存 + mtime 新鲜度检查
3. **向后兼容**: 自动迁移旧配置字段（如 `autoUpdaterStatus` → `installMethod`）
4. **防数据丢失**: 在写入前检查 auth/onboarding 状态
5. **性能监控**: 记录缓存命中率和写入次数
6. **备份机制**: 写入前创建带时间戳的备份

## 与其他文件的关系

| 文件 | 关系 |
|------|------|
| `envUtils.ts` | 使用 `getClaudeConfigHomeDir`, `isEnvTruthy` |
| `errors.ts` | 使用 `ConfigParseError`, `getErrnoCode` |
| `slowOperations.ts` | 使用 `jsonStringify`, `jsonParse` |
| `lockfile.js` | 配置写入锁实现 |
| `cleanupRegistry.ts` | 注册清理回调 |
| `log.ts` | 错误日志记录 |
| `debug.ts` | 调试日志记录 |

## 注意事项

1. **同步 I/O**: 配置读取使用同步文件操作（仅在启动时执行）
2. **测试模式**: `NODE_ENV === 'test'` 时使用测试配置对象
3. **锁竞争**: 锁获取超过100ms会记录调试日志和遥测事件
4. **配置字段**: 新配置字段需要添加到 `GLOBAL_CONFIG_KEYS` 数组
5. **迁移**: 配置结构变更时需要添加迁移逻辑
