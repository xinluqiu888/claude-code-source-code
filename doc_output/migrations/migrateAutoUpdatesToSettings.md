# migrateAutoUpdatesToSettings.ts — 自动更新迁移

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/migrations/migrateAutoUpdatesToSettings.ts`
- **类型**: TypeScript 迁移模块
- **语言**: TypeScript

## 功能概述

将用户设置的 autoUpdates 偏好从全局配置迁移到 settings.json 环境变量。仅迁移用户明确禁用的设置 (非自动保护设置)。

## 核心内容详解

### 主要导出

**migrateAutoUpdatesToSettings()** — 执行迁移

### 迁移条件

1. **全局配置检查**:
   - `autoUpdates` 必须显式为 `false`
   - `autoUpdatesProtectedForNative` 不能为 `true`

2. **排除自动保护**:
   - 仅迁移用户偏好，非自动保护设置

### 迁移流程

1. **获取用户设置**:
   - 读取现有用户设置

2. **设置环境变量**:
   - 设置 `DISABLE_AUTOUPDATER: '1'`
   - 保留其他现有环境变量

3. **立即生效**:
   - 设置 `process.env.DISABLE_AUTOUPDATER = '1'`

4. **记录遥测**:
   - 记录 `tengu_migrate_autoupdates_to_settings`
   - 记录是否为用户偏好和是否已有环境变量

5. **清理全局配置**:
   - 删除 `autoUpdates` 和 `autoUpdatesProtectedForNative`

6. **错误处理**:
   - 记录错误但不中断启动

## 设计要点

1. **用户意图保留**:
   - 仅迁移用户显式禁用的设置
   - 自动保护设置 (native 安装) 不被迁移

2. **环境变量优先**:
   - 迁移到 `DISABLE_AUTOUPDATER` 环境变量
   - 立即生效

3. **幂等性**:
   - 即使环境变量已存在也执行迁移
   - 确保迁移完成

4. **遥测**:
   - 记录迁移事件和状态
   - 帮助跟踪迁移进度

5. **错误恢复**:
   - 错误被记录但不中断
   - 应用继续启动

## 与其他文件的关系

- **../utils/config.ts**: getGlobalConfig, saveGlobalConfig
- **../utils/settings/settings.ts**: getSettingsForSource, updateSettingsForSource
- **../services/analytics/index.ts**: logEvent
- **../utils/log.ts**: logError

## 注意事项

1. 仅迁移用户偏好，非自动保护设置
2. 自动保护设置 (native 安装保护) 被排除
3. 环境变量立即设置生效
4. 即使环境变量已存在也执行迁移
5. 清理后全局配置中的键被删除
