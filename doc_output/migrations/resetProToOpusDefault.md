# resetProToOpusDefault.ts — Pro 用户 Opus 默认重置

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/migrations/resetProToOpusDefault.ts`
- **类型**: TypeScript 迁移模块
- **语言**: TypeScript

## 功能概述

将 Pro 用户重置到 Opus 4.5 默认模型。仅影响使用默认设置 (无自定义模型) 的 Pro 订阅者。

## 核心内容详解

### 主要导出

**resetProToOpusDefault()** — 执行迁移

### 完成标志

- 检查 `opusProMigrationComplete`
- 已完成则跳过

### 迁移条件

1. **API 提供商**:
   - 必须是 'firstParty'

2. **订阅等级**:
   - 必须是 Pro 订阅者

3. **用户设置**:
   - `settings?.model === undefined` (使用默认)

### 迁移流程

**非目标用户**:
- 非第一方 或 非 Pro
- 标记完成并记录跳过

**目标用户** (使用默认):
- 设置 `opusProMigrationTimestamp`
- 标记完成
- 记录事件 (skipped: false, had_custom_model: false)

**自定义模型用户**:
- 仅标记完成
- 记录事件 (skipped: false, had_custom_model: true)

## 设计要点

1. **仅默认用户通知**:
   - 仅当使用默认模型时显示通知
   - 自定义模型用户不显示

2. **时间戳用途**:
   - `opusProMigrationTimestamp` 用于通知用户体验
   - 仅在需要通知的用户设置

3. **完成标志**:
   - 所有路径都设置 `opusProMigrationComplete`
   - 确保迁移只运行一次

4. **遥测**:
   - 记录 `tengu_reset_pro_to_opus_default`
   - 记录跳过状态和自定义模型状态

5. **旧版设置 API**:
   - 使用 `getSettings_DEPRECATED()`
   - 可能需要迁移到新版 API

## 与其他文件的关系

- **../services/analytics/index.ts**: logEvent
- **../utils/auth.ts**: isProSubscriber
- **../utils/config.ts**: getGlobalConfig, saveGlobalConfig
- **../utils/model/providers.ts**: getAPIProvider
- **../utils/settings/settings.ts**: getSettings_DEPRECATED

## 注意事项

1. 仅第一方 Pro 订阅者
2. 仅使用默认模型的用户获得时间戳
3. 自定义模型用户被标记完成但不获得时间戳
4. 遥测帮助验证迁移覆盖
5. 使用旧版设置 API 可能需要更新
