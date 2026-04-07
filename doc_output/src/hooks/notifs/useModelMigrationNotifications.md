# useModelMigrationNotifications.tsx — 模型迁移通知

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useModelMigrationNotifications.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `useModelMigrationNotifications`
- **依赖**: useStartupNotification, GlobalConfig

## 功能概述

本 Hook 在模型迁移完成后显示一次性通知，告知用户模型已更新。支持 Sonnet 4.5→4.6 和 Opus 迁移。

## 核心内容详解

### 迁移定义

```typescript
const MIGRATIONS: ((c: GlobalConfig) => Notification | undefined)[] = [
  // Sonnet 4.5 → 4.6
  c => {
    if (!recent(c.sonnet45To46MigrationTimestamp)) return
    return {
      key: 'sonnet-46-update',
      text: 'Model updated to Sonnet 4.6',
      color: 'suggestion',
      priority: 'high',
      timeoutMs: 3000,
    }
  },
  
  // Opus Pro → default, 或 pinned 4.0/4.1 → opus alias
  c => {
    const isLegacyRemap = Boolean(c.legacyOpusMigrationTimestamp)
    const ts = c.legacyOpusMigrationTimestamp ?? c.opusProMigrationTimestamp
    if (!recent(ts)) return
    return {
      key: 'opus-pro-update',
      text: isLegacyRemap 
        ? 'Model updated to Opus 4.6 · Set CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP=1 to opt out'
        : 'Model updated to Opus 4.6',
      color: 'suggestion',
      priority: 'high',
      timeoutMs: isLegacyRemap ? 8000 : 3000,
    }
  },
]
```

### 时间窗口判断

```typescript
function recent(ts: number | undefined): boolean {
  return ts !== undefined && Date.now() - ts < 3000
}
```

仅当迁移时间戳在 3 秒内（即本次启动期间）时才显示通知。

### 执行流程

```typescript
useStartupNotification(() => {
  const config = getGlobalConfig()
  const notifs = []
  for (const migration of MIGRATIONS) {
    const notif = migration(config)
    if (notif) {
      notifs.push(notif)
    }
  }
  return notifs.length > 0 ? notifs : null
})
```

## 设计要点

### 1. 时间窗口

仅在本会话期间发生的迁移才显示通知，避免每次启动都显示。

### 2. 扩展性

通过 `MIGRATIONS` 数组定义迁移，新增迁移只需添加新条目。

### 3. 历史兼容

支持区分传统重映射（legacy remap）和普通迁移，显示不同提示。

### 4. 批量通知

支持同时显示多条迁移通知（如多个模型同时迁移）。

### 5. 退出提示

传统重映射显示环境变量退出方式，给用户选择权。

## 与其他文件的关系

- **useStartupNotification.ts**: 基础 Hook
- **config.ts**: 全局配置读取

## 注意事项

1. **时间精度**: 依赖时间戳，系统时间变化可能影响判断
2. **启动时显示**: 仅在启动时检查，运行期间发生的迁移不会立即显示
3. **多迁移**: 理论上支持同时显示多个迁移通知
4. **超时差异**: 传统重映射超时更长（8秒 vs 3秒），因为需要阅读退出说明
