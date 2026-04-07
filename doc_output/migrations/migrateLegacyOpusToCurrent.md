# migrateLegacyOpusToCurrent.ts — 旧版 Opus 到当前版本迁移

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/migrations/migrateLegacyOpusToCurrent.ts`
- **类型**: TypeScript 迁移模块
- **语言**: TypeScript

## 功能概述

将第一方用户从显式 Opus 4.0/4.1 模型字符串迁移到 'opus' 别名 (现在解析为 Opus 4.6)。

## 核心内容详解

### 主要导出

**migrateLegacyOpusToCurrent()** — 执行迁移

### 迁移条件

1. **API 提供商**:
   - 必须是 'firstParty'

2. **特性门控**:
   - `isLegacyModelRemapEnabled()` 必须返回 true

3. **模型匹配**:
   - `claude-opus-4-20250514`
   - `claude-opus-4-1-20250805`
   - `claude-opus-4-0`
   - `claude-opus-4-1`

### 迁移流程

1. **更新用户设置**:
   - 设置 `model: 'opus'`
   - 使用 `updateSettingsForSource('userSettings', ...)`

2. **设置时间戳**:
   - 在全局配置中设置 `legacyOpusMigrationTimestamp`
   - 用于 REPL 一次性通知

3. **遥测记录**:
   - 记录 `tengu_legacy_opus_migration`
   - 记录源模型

## 设计要点

1. **仅用户设置**:
   - 仅读取/写入 userSettings
   - 项目/本地/策略设置中的旧字符串保持不变

2. **运行时重映射**:
   - `parseUserSpecifiedModel` 已运行时重映射旧字符串
   - 此迁移清理设置文件

3. **时间戳用途**:
   - REPL 使用 `legacyOpusMigrationTimestamp` 显示一次性通知
   - 仅在需要时通知用户

4. **幂等性**:
   - 读取和写入同一源保持幂等
   - 无需完成标志

5. **不提升全局默认**:
   - 不将 'opus' 提升为全局默认
   - 仅替换用户显式设置的值

## 与其他文件的关系

- **../utils/config.ts**: saveGlobalConfig
- **../utils/model/model.ts**: isLegacyModelRemapEnabled
- **../utils/model/providers.ts**: getAPIProvider
- **../utils/settings/settings.ts**: getSettingsForSource, updateSettingsForSource
- **../services/analytics/index.ts**: logEvent

## 注意事项

1. 项目/本地/策略设置中的旧字符串不被重写
2. 运行时 `parseUserSpecifiedModel` 仍会重映射这些字符串
3. 仅影响显式设置旧字符串的用户
4. 遥测帮助跟踪迁移效果
5. 时间戳用于 REPL 通知用户体验
