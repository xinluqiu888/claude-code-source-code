# format.ts — 格式化工具函数

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/format.ts`
- **主要功能**: 各种数据格式化函数（文件大小、持续时间、数字、相对时间）
- **关键依赖**: `intl.ts`（国际化支持）, `truncate.ts`

## 功能概述

该模块提供纯显示格式化工具函数：
1. 文件大小格式化（字节 → KB/MB/GB）
2. 持续时间格式化（毫秒 → 可读格式）
3. 数字格式化（千分位、紧凑格式）
4. 相对时间格式化（X天前、Y分钟后）
5. 日志元数据格式化
6. 重置时间格式化

## 核心内容详解

### 文件大小格式化

```typescript
export function formatFileSize(sizeInBytes: number): string
// 示例: 1536 -> "1.5KB"
```

### 持续时间格式化

#### `formatSecondsShort(ms): string`
- 格式化为秒，保留1位小数（如 `1234` → `"1.2s"`）
- 用于短时计时（TTFT、hook 持续时间）

#### `formatDuration(ms, options?): string`
- 完整持续时间格式化
- 支持 `hideTrailingZeros` 和 `mostSignificantOnly` 选项
- 处理进位（59.5秒 → 1分钟）

```typescript
formatDuration(90000)        // "1m 30s"
formatDuration(86400000 * 2) // "2d 0h 0m"
```

### 数字格式化

```typescript
export function formatNumber(number: number): string
// 示例: 1321 -> "1.3k", 900 -> "900"
```

使用 `Intl.NumberFormat` 缓存优化性能。

### 相对时间格式化

```typescript
export function formatRelativeTime(date, options?): string
export function formatRelativeTimeAgo(date, options?): string
// 示例: formatRelativeTimeAgo(new Date(Date.now() - 86400000)) -> "1d ago"
```

支持 `long`/`short`/`narrow` 样式和 `always`/`auto` 数字模式。

### 日志元数据格式化

```typescript
export function formatLogMetadata(log): string
// 示例: "2d ago · main · 1.5MB · #tag"
```

组合显示：相对时间、Git分支、大小/消息数、标签、PR号等。

### 重置时间格式化

```typescript
export function formatResetTime(timestamp, showTimezone?, showTime?): string
// 示例: "3:30pm (PST)"
```

- 24小时内只显示时间
- 超过24小时显示日期和时间

## 设计要点

1. **国际化**: 使用 `Intl` API 支持多语言
2. **性能优化**: 缓存 `NumberFormat` 实例
3. **精度控制**: 持续时间处理边界情况（进位、零值）
4. **向后兼容**: 截断函数从本模块移至 `truncate.ts`

## 与其他文件的关系

| 文件 | 关系 |
|------|------|
| `intl.ts` | 导入 `getRelativeTimeFormat`, `getTimeZone` |
| `truncate.ts` | 重新导出截断函数（向后兼容） |

## 注意事项

1. **纯显示**: 这些函数仅用于显示，不改变底层数据
2. **Intl 缓存**: `NumberFormat` 实例按配置缓存
3. **时区**: `formatResetTime` 使用本地时区或显式显示
