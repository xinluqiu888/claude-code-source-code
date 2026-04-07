# migrateSonnet1mToSonnet45.ts — Sonnet 1M 到 Sonnet 4.5 迁移

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/migrations/migrateSonnet1mToSonnet45.ts`
- **类型**: TypeScript 迁移模块
- **语言**: TypeScript

## 功能概述

将用户从 "sonnet[1m]" 迁移到显式 "sonnet-4-5-20250929[1m]"。"sonnet" 别名现在解析为 Sonnet 4.6，需要固定现有 sonnet[1m] 用户到 Sonnet 4.5 1M 以保留其预期模型。

## 核心内容详解

### 主要导出

**migrateSonnet1mToSonnet45()** — 执行迁移

### 迁移条件

1. **完成标志**:
   - 检查 `sonnet1m45MigrationComplete`
   - 已完成则跳过

2. **模型匹配**:
   - 必须是 `'sonnet[1m]'`

### 迁移流程

1. **更新用户设置**:
   - 设置 `model: 'sonnet-4-5-20250929[1m]'`
   - 使用 `updateSettingsForSource('userSettings', ...)`

2. **更新内存覆盖**:
   - 检查 `getMainLoopModelOverride()`
   - 如果是 `'sonnet[1m]'`，更新为 `'sonnet-4-5-20250929[1m]'`

3. **设置完成标志**:
   - 设置 `sonnet1m45MigrationComplete: true`

## 设计要点

1. **显式版本固定**:
   - 从别名迁移到显式版本
   - 保留用户对特定版本的预期

2. **仅用户设置**:
   - 仅读取/写入 userSettings
   - 不修改项目/本地设置

3. **内存覆盖**:
   - 同时检查并更新 `getMainLoopModelOverride`
   - 防止运行时覆盖冲突

4. **完成标志**:
   - 全局配置中的 `sonnet1m45MigrationComplete`
   - 确保迁移只运行一次

5. **必要性**:
   - Sonnet 4.6 1M 向不同于 Sonnet 4.5 1M 的用户群提供
   - 需要固定现有用户到其预期的模型

## 与其他文件的关系

- **../bootstrap/state.ts**: getMainLoopModelOverride, setMainLoopModelOverride
- **../utils/config.ts**: getGlobalConfig, saveGlobalConfig
- **../utils/settings/settings.ts**: getSettingsForSource, updateSettingsForSource

## 注意事项

1. 仅针对显式设置 'sonnet[1m]' 的用户
2. 迁移到显式版本字符串而非别名
3. 同时更新设置和内存覆盖
4. 完成标志确保幂等性
5. 这是向前兼容迁移，非向后兼容
