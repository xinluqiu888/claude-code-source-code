# tipHistory.ts — 提示历史记录

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/tips/tipHistory.ts`
- **作用域**: 提示显示历史记录和会话统计
- **主要导出**:
  - `recordTipShown`: 记录提示已显示
  - `getSessionsSinceLastShown`: 获取自上次显示以来的会话数

## 功能概述

管理提示的显示历史。使用全局配置存储每个提示的最后显示会话编号，通过对比当前会话数计算冷却状态。

## 核心内容详解

### 数据结构

提示历史存储在全局配置的 `tipsHistory` 字段：
```typescript
{
  tipsHistory: {
    [tipId: string]: number  // 提示ID -> 显示时的会话编号
  }
}
```

### 核心函数

#### `recordTipShown(tipId)`
记录提示已显示：

**参数**:
- `tipId`: 提示唯一标识

**流程**:
1. 获取当前会话数 (`numStartups`)
2. 如果该提示已在当前会话记录过，跳过
3. 否则更新全局配置，记录 `tipId -> numStartups`

**实现**:
```typescript
export function recordTipShown(tipId: string): void {
  const numStartups = getGlobalConfig().numStartups
  saveGlobalConfig(c => {
    const history = c.tipsHistory ?? {}
    if (history[tipId] === numStartups) return c  // 已记录，跳过
    return { ...c, tipsHistory: { ...history, [tipId]: numStartups } }
  })
}
```

#### `getSessionsSinceLastShown(tipId)`
获取自上次显示以来的会话数：

**参数**:
- `tipId`: 提示唯一标识

**返回值**:
- `number`: 自上次显示以来的会话数
- `Infinity`: 如果从未显示过

**计算逻辑**:
```typescript
export function getSessionsSinceLastShown(tipId: string): number {
  const config = getGlobalConfig()
  const lastShown = config.tipsHistory?.[tipId]
  if (!lastShown) return Infinity
  return config.numStartups - lastShown
}
```

## 设计要点

1. **会话计数**: 使用 `numStartups`（会话启动次数）而非时间戳
2. **去重机制**: 同一提示在同一会话中只记录一次
3. **不可变更新**: 使用函数式更新方式更新全局配置
4. **默认值处理**: 未找到记录时返回 `Infinity`，确保新提示优先显示

## 与其他文件的关系

- **tipScheduler.ts**: 调用 `recordTipShown` 和 `getSessionsSinceLastShown`
- **utils/config.ts**: 使用 `getGlobalConfig` 和 `saveGlobalConfig`

## 注意事项

1. **持久化**: 历史记录保存在全局配置文件中
2. **会话粒度**: 基于会话而非时间，重启后计数继续
3. **内存效率**: 仅存储提示ID和会话编号，数据量小
