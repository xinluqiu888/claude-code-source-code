# migrateSonnet45ToSonnet46.ts — Sonnet 4.5 到 4.6 模型迁移

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/migrations/migrateSonnet45ToSonnet46.ts`
- **类型**: TypeScript 迁移模块
- **语言**: TypeScript

## 功能概述

将 Pro/Max/Team Premium 第一方用户从显式 Sonnet 4.5 模型字符串迁移到 'sonnet' 别名 (现在解析为 Sonnet 4.6)。

## 核心内容详解

### 主要导出

**migrateSonnet45ToSonnet46()** — 执行迁移

### 迁移条件

1. **API 提供商**:
   - 必须是 'firstParty'
   - 第三方用户被跳过

2. **订阅等级**:
   - Pro、Max 或 Team Premium 订阅者
   - 非订阅者被跳过

3. **模型匹配**:
   - `claude-sonnet-4-5-20250929`
   - `claude-sonnet-4-5-20250929[1m]`
   - `sonnet-4-5-20250929`
   - `sonnet-4-5-20250929[1m]`

### 迁移流程

1. **检查 1M 后缀**:
   - 检测模型是否以 `[1m]` 结尾

2. **更新设置**:
   - 有 [1m]: 设置为 `'sonnet[1m]'`
   - 无 [1m]: 设置为 `'sonnet'`
   - 使用 `updateSettingsForSource('userSettings', ...)`

3. **通知标记**:
   - 检查 `numStartups > 1` (非新用户)
   - 设置 `sonnet45To46MigrationTimestamp`

4. **遥测记录**:
   - 记录 `tengu_sonnet45_to_46_migration`
   - 记录源模型和 1M 状态

## 设计要点

1. **仅用户设置**:
   - 仅读取/写入 userSettings
   - 不修改项目/本地设置

2. **别名解析**:
   - 使用 'sonnet' 别名而非显式版本
   - 别名自动解析到最新版本

3. **1M 保留**:
   - 检测并保留 [1m] 后缀
   - 区分标准版和 1M 上下文版

4. **新用户跳过**:
   - `numStartups > 1` 检查
   - 首次启动用户不显示通知

5. **幂等性**:
   - 仅匹配特定模型字符串
   - 迁移后字符串不再匹配

## 与其他文件的关系

- **../utils/auth.ts**: isProSubscriber, isMaxSubscriber, isTeamPremiumSubscriber
- **../utils/config.ts**: getGlobalConfig, saveGlobalConfig
- **../utils/settings/settings.ts**: getSettingsForSource, updateSettingsForSource
- **../utils/model/providers.ts**: getAPIProvider
- **../services/analytics/index.ts**: logEvent

## 注意事项

1. 仅影响第一方 API 用户
2. 第三方用户使用完整模型 ID，不是别名
3. CLI 的 `--model opus` 不受影响 (运行时覆盖)
4. Pro 用户保留单独的 Opus 和 Opus 1M 选项
5. 遥测帮助跟踪迁移覆盖范围
