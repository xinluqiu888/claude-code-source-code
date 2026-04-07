# migrateEnableAllProjectMcpServersToSettings.ts — MCP 服务器启用迁移

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/migrations/migrateEnableAllProjectMcpServersToSettings.ts`
- **类型**: TypeScript 迁移模块
- **语言**: TypeScript

## 功能概述

将 MCP 服务器批准字段从项目配置迁移到本地设置。迁移 `enableAllProjectMcpServers`、`enabledMcpjsonServers` 和 `disabledMcpjsonServers`。

## 核心内容详解

### 主要导出

**migrateEnableAllProjectMcpServersToSettings()** — 执行迁移

### 迁移字段

1. **enableAllProjectMcpServers** — 启用所有项目 MCP 服务器
2. **enabledMcpjsonServers** — 启用的 MCP 服务器列表
3. **disabledMcpjsonServers** — 禁用的 MCP 服务器列表

### 迁移流程

1. **检查源字段**:
   - 检查项目配置中是否存在任何字段
   - 都不存在则提前返回

2. **获取现有设置**:
   - 读取 `localSettings` 现有设置

3. **准备更新**:
   - `updates`: 要更新的字段
   - `fieldsToRemove`: 要从项目配置移除的字段

4. **迁移每个字段**:
   - **enableAllProjectMcpServers**:
     - 检查本地设置是否未定义
     - 复制值到 updates
     - 标记为移除
   
   - **enabledMcpjsonServers**:
     - 合并现有和新列表 (去重)
     - 复制到 updates
     - 标记为移除
   
   - **disabledMcpjsonServers**:
     - 合并现有和新列表 (去重)
     - 复制到 updates
     - 标记为移除

5. **更新本地设置**:
   - 如果有更新则调用 `updateSettingsForSource('localSettings', ...)`

6. **清理项目配置**:
   - 使用 `saveCurrentProjectConfig` 移除字段

7. **遥测记录**:
   - 记录成功或错误事件

8. **错误处理**:
   - 记录失败但不中断启动

## 设计要点

1. **合并策略**:
   - 服务器列表合并并去重
   - 保留现有和新条目

2. **设置分层**:
   - 迁移到 `localSettings`
   - 项目级设置 → 本地设置

3. **幂等性**:
   - `enableAllProjectMcpServers` 检查是否已迁移
   - 服务器列表合并处理重复

4. **遥测**:
   - 记录 `tengu_migrate_mcp_approval_fields_success`
   - 记录 `tengu_migrate_mcp_approval_fields_error`

5. **错误恢复**:
   - 失败不中断启动
   - 记录供调试

## 与其他文件的关系

- **../utils/config.ts**: getCurrentProjectConfig, saveCurrentProjectConfig
- **../utils/settings/settings.ts**: getSettingsForSource, updateSettingsForSource
- **../services/analytics/index.ts**: logEvent
- **../utils/log.ts**: logError

## 注意事项

1. 仅迁移存在的字段
2. 服务器列表合并并去重
3. `enableAllProjectMcpServers` 检查是否已存在
4. 错误不中断应用启动
5. 遥测帮助验证迁移成功
