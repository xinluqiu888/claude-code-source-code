# migrateOpusToOpus1m.ts — Opus 到 Opus 1M 迁移

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/migrations/migrateOpusToOpus1m.ts`
- **类型**: TypeScript 迁移模块
- **语言**: TypeScript

## 功能概述

将设置中固定为 'opus' 的用户在有资格获得合并的 Opus 1M 体验时迁移到 'opus[1m]'。

## 核心内容详解

### 主要导出

**migrateOpusToOpus1m()** — 执行迁移

### 迁移条件

1. **特性门控**:
   - `isOpus1mMergeEnabled()` 必须返回 true

2. **模型匹配**:
   - 必须是 `'opus'`

3. **订阅等级**:
   - Max 或 Team Premium 订阅者
   - Pro 订阅者被跳过

### 迁移流程

1. **确定目标模型**:
   - 目标: `'opus[1m]'`

2. **智能设置**:
   - 如果 `parseUserSpecifiedModel(migrated) === parseUserSpecifiedModel(getDefaultMainLoopModelSetting())`
   - 则设置为 `undefined` (使用默认)
   - 否则设置为 `'opus[1m]'`

3. **更新设置**:
   - 使用 `updateSettingsForSource('userSettings', { model: modelToSet })`

4. **遥测记录**:
   - 记录 `tengu_opus_to_opus1m_migration`

## 设计要点

1. **1M 合并体验**:
   - 合并 Opus 和 Opus 1M 为单一选项
   - 1M 上下文成为默认

2. **Pro 用户排除**:
   - Pro 订阅者保留单独的 Opus 和 Opus 1M 选项
   - Max/Team Premium 获得合并体验

3. **智能默认**:
   - 如果迁移后模型等于新默认，清除设置
   - 避免不必要的显式设置

4. **仅用户设置**:
   - 仅读取/写入 userSettings
   - `--model opus` CLI 标志不受影响

5. **运行时覆盖**:
   - CLI 标志是运行时覆盖
   - 不触及设置

## 与其他文件的关系

- **../utils/model/model.ts**: isOpus1mMergeEnabled, parseUserSpecifiedModel, getDefaultMainLoopModelSetting
- **../utils/settings/settings.ts**: getSettingsForSource, updateSettingsForSource
- **../services/analytics/index.ts**: logEvent

## 注意事项

1. CLI `--model opus` 标志不受影响
2. 仅迁移显式设置 'opus' 的用户
3. Pro 订阅者被排除在合并体验外
4. 智能检查避免冗余显式设置
5. 遥测帮助验证迁移范围
