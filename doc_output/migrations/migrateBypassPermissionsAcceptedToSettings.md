# migrateBypassPermissionsAcceptedToSettings.ts — 迁移绕过权限接受设置

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/migrations/migrateBypassPermissionsAcceptedToSettings.ts`
- **类型**: TypeScript 迁移模块
- **语言**: TypeScript

## 功能概述

将 `bypassPermissionsModeAccepted` 从全局配置迁移到 settings.json 中的 `skipDangerousModePermissionPrompt`。将设置迁移到用户可配置的位置。

## 核心内容详解

### 主要导出

**migrateBypassPermissionsAcceptedToSettings()** — 执行迁移

### 迁移流程

1. **检查源设置**:
   - 获取全局配置
   - 检查 `bypassPermissionsModeAccepted` 是否存在
   - 不存在则提前返回

2. **迁移到用户设置**:
   - 检查 `skipDangerousModePermissionPrompt` 是否已设置
   - 未设置则更新用户设置为 `true`
   - 使用 `updateSettingsForSource('userSettings', ...)`

3. **记录遥测**:
   - 记录 `tengu_migrate_bypass_permissions_accepted` 事件

4. **清理源设置**:
   - 从全局配置中删除 `bypassPermissionsModeAccepted`
   - 使用 `saveGlobalConfig` 更新

5. **错误处理**:
   - 捕获并记录迁移错误
   - 不中断应用启动

## 设计要点

1. **幂等性**:
   - 多次运行安全
   - 检查目标设置是否已存在
   - 清理后源设置不再存在

2. **设置分层**:
   - 从全局配置迁移到用户设置
   - 更好的用户可配置性

3. **遥测记录**:
   - 记录迁移事件
   - 帮助跟踪迁移进度

4. **错误恢复**:
   - 错误不中断启动
   - 记录供调试

## 与其他文件的关系

- **../utils/config.ts**: getGlobalConfig, saveGlobalConfig
- **../utils/settings/settings.ts**: updateSettingsForSource, hasSkipDangerousModePermissionPrompt
- **../services/analytics/index.ts**: logEvent
- **../utils/log.ts**: logError

## 注意事项

1. 迁移后全局配置中的旧键被删除
2. 目标设置已存在时跳过迁移
3. 错误被记录但不中断应用
4. 属于一次性迁移，执行后不再需要
