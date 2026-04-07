# resetAutoModeOptInForDefaultOffer.ts — 重置自动模式选择

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/migrations/resetAutoModeOptInForDefaultOffer.ts`
- **类型**: TypeScript 迁移模块
- **语言**: TypeScript

## 功能概述

一次性迁移：为接受旧版 2 选项 AutoModeOptInDialog 但没有自动模式作为默认的用户清除 `skipAutoPermissionPrompt`。让他们看到新的"设为默认模式"选项。

## 核心内容详解

### 主要导出

**resetAutoModeOptInForDefaultOffer()** — 执行迁移

### 迁移条件

1. **特性门控**:
   - 需要 `TRANSCRIPT_CLASSIFIER` 特性启用

2. **全局配置检查**:
   - 检查 `hasResetAutoModeOptInForDefaultOffer`
   - 已设置则跳过

3. **自动模式状态**:
   - 检查 `getAutoModeEnabledState() === 'enabled'`
   - 不是 'enabled' 则跳过 (防止 opt-in 用户失去访问)

### 迁移流程

1. **获取用户设置**:
   - 检查 `skipAutoPermissionPrompt` 是否为 true
   - 检查 `permissions.defaultMode` 是否不是 'auto'

2. **执行重置**:
   - 清除 `skipAutoPermissionPrompt`
   - 设置为 `undefined`
   - 记录遥测事件

3. **设置完成标志**:
   - 在全局配置中设置 `hasResetAutoModeOptInForDefaultOffer: true`

4. **错误处理**:
   - 捕获并记录错误
   - 不中断启动

## 设计要点

1. **一次性迁移**:
   - 全局配置中的完成标志
   - 迁移后不再运行

2. **安全门控**:
   - 仅影响 'enabled' 状态用户
   - 'opt-in' 用户被保护

3. **目标用户**:
   - 约 40 个目标蚂蚁用户
   - 通过 Shift+Tab 访问旧对话框的用户

4. **设置分层**:
   - 仅修改用户设置
   - 不影响项目/本地设置

5. **遥测**:
   - 记录 `tengu_migrate_reset_auto_opt_in_for_default_offer`
   - 跟踪迁移进度

## 与其他文件的关系

- **../utils/config.ts**: getGlobalConfig, saveGlobalConfig
- **../utils/settings/settings.ts**: getSettingsForSource, updateSettingsForSource
- **../utils/permissions/permissionSetup.ts**: getAutoModeEnabledState
- **../services/analytics/index.ts**: logEvent

## 注意事项

1. 全局配置门控确保迁移只运行一次
2. 'opt-in' 用户被保护不失去对话框访问
3. 仅影响通过 Shift+Tab 访问旧对话框的用户
4. 错误不中断应用启动
5. 遥测帮助验证迁移效果
