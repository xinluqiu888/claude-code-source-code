# migrateFennecToOpus.ts — Fennec 到 Opus 模型迁移

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/migrations/migrateFennecToOpus.ts`
- **类型**: TypeScript 迁移模块
- **语言**: TypeScript

## 功能概述

将使用已移除 Fennec 模型别名的用户迁移到新的 Opus 4.6 别名。

## 核心内容详解

### 主要导出

**migrateFennecToOpus()** — 执行迁移

### 用户类型检查

- 仅 `process.env.USER_TYPE === 'ant'` 用户执行
- 非 Ant 用户被跳过

### 迁移映射

| 源模型 | 目标模型 | 额外设置 |
|--------|---------|---------|
| fennec-latest[1m] | opus[1m] | 无 |
| fennec-latest | opus | 无 |
| fennec-fast-latest | opus[1m] | fastMode: true |
| opus-4-5-fast | opus[1m] | fastMode: true |

### 迁移流程

1. **获取用户设置**:
   - 读取 `userSettings`

2. **检查模型**:
   - 如果模型是字符串类型
   - 匹配各种 Fennec 别名模式

3. **执行迁移**:
   - 根据匹配的模式设置新模型
   - 如果需要设置 `fastMode: true`
   - 使用 `updateSettingsForSource('userSettings', ...)`

4. **模式匹配**:
   - 使用 `startsWith` 进行前缀匹配
   - 处理 [1m] 变体

## 设计要点

1. **仅用户设置**:
   - 仅修改 `userSettings`
   - 不修改项目/本地/策略设置

2. **Fast 模式处理**:
   - Fennec fast 变体映射到 Opus 1M + fastMode
   - 保持快速模式行为

3. **前缀匹配**:
   - 使用 `startsWith` 匹配
   - 处理带版本后缀的别名

4. **无遥测**:
   - 简单别名重映射
   - 无事件记录

5. **无完成标志**:
   - 读取和写入同一源
   - 幂等无需标志

6. **项目设置保留**:
   - 项目/本地/策略设置中的 Fennec 别名保持不变
   - 运行时可能仍有重映射

## 与其他文件的关系

- **../utils/settings/settings.ts**: getSettingsForSource, updateSettingsForSource

## 注意事项

1. 仅 Ant 用户执行迁移
2. 项目/本地/策略设置中的别名不修改
3. Fennec fast 变体变为 Opus 1M + fastMode
4. 使用前缀匹配处理版本后缀
5. 无遥测或完成标志的简单迁移
