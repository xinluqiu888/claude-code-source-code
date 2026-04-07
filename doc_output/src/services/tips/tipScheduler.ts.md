# tipScheduler.ts — 提示调度器

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/tips/tipScheduler.ts`
- **作用域**: 旋转提示的调度和选择逻辑
- **主要导出**:
  - `selectTipWithLongestTimeSinceShown`: 选择最久未显示的提示
  - `getTipToShowOnSpinner`: 获取要在旋转器上显示的提示
  - `recordShownTip`: 记录已显示的提示

## 功能概述

管理 Claude Code 旋转提示的调度逻辑。根据提示的冷却时间和历史显示记录，选择最合适的提示在加载旋转器上展示。

## 核心内容详解

### 类型定义（推断）

```typescript
type Tip = {
  id: string                           // 提示唯一标识
  content: (ctx?: TipContext) => Promise<string>  // 提示内容生成函数
  cooldownSessions: number             // 冷却会话数
  isRelevant?: (ctx?: TipContext) => Promise<boolean>  // 相关性检查（可选）
}

type TipContext = {
  bashTools?: Set<string>              // 使用的bash工具
  readFileState?: FileStateCache       // 读取文件状态
  theme?: string                       // 主题
}
```

### 核心函数

#### `selectTipWithLongestTimeSinceShown(availableTips)`
选择最久未显示的提示：

**逻辑**:
1. 将提示映射为包含会话数的数据结构
2. 按 `sessions` 降序排序
3. 返回排序后第一个提示

**算法**:
```typescript
const tipsWithSessions = availableTips.map(tip => ({
  tip,
  sessions: getSessionsSinceLastShown(tip.id),
}))
tipsWithSessions.sort((a, b) => b.sessions - a.sessions)
```

#### `getTipToShowOnSpinner(context?)`
获取要在旋转器上显示的提示：

**参数**:
- `context`: 可选的提示上下文

**流程**:
1. 检查设置中是否禁用提示 (`spinnerTipsEnabled`)
2. 调用 `getRelevantTips(context)` 获取相关提示
3. 调用 `selectTipWithLongestTimeSinceShown` 选择提示

#### `recordShownTip(tip)`
记录已显示的提示：

**操作**:
1. 调用 `recordTipShown(tip.id)` 记录到历史
2. 记录遥测事件 `tengu_tip_shown`

### 遥测事件

**tengu_tip_shown**:
- `tipIdLength`: 提示ID长度
- `cooldownSessions`: 冷却会话数

## 设计要点

1. **冷却机制**: 基于会话数的冷却，避免频繁显示同一提示
2. **相关性过滤**: 只显示与用户当前上下文相关的提示
3. **历史追踪**: 记录每个提示的最后显示时间
4. **可禁用**: 用户可通过设置关闭提示

## 与其他文件的关系

- **tipHistory.ts**: 提供 `getSessionsSinceLastShown`, `recordTipShown`
- **tipRegistry.ts**: 提供 `getRelevantTips`
- **analytics/index.ts**: 提供 `logEvent`

## 注意事项

1. **设置检查**: 使用 `getSettings_DEPRECATED()` 检查 `spinnerTipsEnabled`
2. **空值处理**: 当没有可用提示时返回 `undefined`
3. **排序稳定性**: 使用 `sort` 进行降序排序
