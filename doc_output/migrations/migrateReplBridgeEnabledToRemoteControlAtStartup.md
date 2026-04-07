# migrateReplBridgeEnabledToRemoteControlAtStartup.ts — REPL 桥接启用迁移

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/migrations/migrateReplBridgeEnabledToRemoteControlAtStartup.ts`
- **类型**: TypeScript 迁移模块
- **语言**: TypeScript

## 功能概述

将 `replBridgeEnabled` 配置键迁移到 `remoteControlAtStartup`。旧键是实现细节泄露到了用户配置中。

## 核心内容详解

### 主要导出

**migrateReplBridgeEnabledToRemoteControlAtStartup()** — 执行迁移

### 迁移流程

1. **检查旧键**:
   - 通过未类型化访问 `prev['replBridgeEnabled']`
   - 不存在则返回原配置

2. **检查新键**:
   - 检查 `prev.remoteControlAtStartup`
   - 已存在则返回原配置 (幂等)

3. **执行迁移**:
   - 复制旧值到新键: `remoteControlAtStartup: Boolean(oldValue)`
   - 删除旧键

4. **保存配置**:
   - 使用 `saveGlobalConfig` 更新

## 设计要点

1. **键重命名**:
   - 从 `replBridgeEnabled` 到 `remoteControlAtStartup`
   - 更好的用户可理解性

2. **未类型化访问**:
   - 旧键已从 GlobalConfig 类型中移除
   - 使用 `as Record<string, unknown>` 访问

3. **幂等性**:
   - 新键存在时跳过
   - 旧键不存在时跳过

4. **布尔转换**:
   - 使用 `Boolean(oldValue)` 确保布尔值

5. **无遥测**:
   - 简单键重命名
   - 无需遥测跟踪

## 与其他文件的关系

- **../utils/config.ts**: saveGlobalConfig

## 注意事项

1. 旧键已从 GlobalConfig 类型中移除
2. 需要未类型化访问读取旧值
3. 新键不存在时才执行迁移
4. 迁移后旧键被删除
5. 简单键重命名，无复杂逻辑
