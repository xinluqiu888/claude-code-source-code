# theme.ts — 主题颜色和颜色方案

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/theme.ts`
- **主要功能**: 定义主题颜色系统，支持多种主题模式
- **关键依赖**: `chalk`, `env.ts`

## 功能概述

该模块定义 Claude Code 的完整主题颜色系统：
1. 6种预定义主题（dark/light/daltonized/ansi）
2. 语义化颜色变量（success/error/warning 等）
3. TUI V2 专用颜色
4. Agent 专用颜色
5. 颜色转换工具

## 核心内容详解

### 主题类型定义

```typescript
export type Theme = {
  // 核心品牌色
  claude: string                    // Claude 橙色
  claudeShimmer: string             // 浅色 shimmer 效果
  claudeBlue_FOR_SYSTEM_SPINNER: string

  // UI 边框/背景
  bashBorder: string
  promptBorder: string
  userMessageBackground: string
  messageActionsBackground: string
  selectionBg: string

  // 语义颜色
  success: string
  error: string
  warning: string
  merged: string

  // Diff 颜色
  diffAdded: string
  diffRemoved: string
  diffAddedWord: string
  diffRemovedWord: string

  // Agent 颜色（仅子代理使用）
  red_FOR_SUBAGENTS_ONLY: string
  blue_FOR_SUBAGENTS_ONLY: string
  // ... 更多

  // 彩虹色（ultrathink 关键词高亮）
  rainbow_red: string
  rainbow_orange: string
  // ... 更多
}
```

### 主题列表

```typescript
export const THEME_NAMES = [
  'dark',           // 默认暗色主题
  'light',          // 亮色主题
  'light-daltonized',  // 色盲友好亮色
  'dark-daltonized',   // 色盲友好暗色
  'light-ansi',     // 仅 ANSI 颜色亮色
  'dark-ansi',      // 仅 ANSI 颜色暗色
] as const
```

### 主题获取

```typescript
export function getTheme(themeName: ThemeName): Theme
```

- 根据主题名称返回完整颜色配置
- 无效名称返回 `dark` 主题

### 颜色转换

```typescript
export function themeColorToAnsi(themeColor: string): string
```

- 将 RGB 主题颜色转换为 ANSI 转义序列
- 支持 Apple Terminal 的 256 色模式降级

## 设计要点

1. **色盲友好**: daltonized 主题针对色盲用户优化
2. **ANSI 降级**: ansi 主题支持不支持真彩色的终端
3. **语义化命名**: 颜色按用途命名而非颜色名
4. **shimmer 变体**: 每个主色都有对应的 shimmer（浅色）变体
5. **Apple Terminal 兼容**: 检测 Apple Terminal 使用 256 色模式

## 与其他文件的关系

| 文件 | 关系 |
|------|------|
| `chalk` | 颜色处理和 ANSI 生成 |
| `env.ts` | 检测终端类型（Apple_Terminal） |

## 注意事项

1. **颜色格式**: 支持 `rgb(r,g,b)` 和 `ansi:colorname` 格式
2. **Agent 颜色**: `*_FOR_SUBAGENTS_ONLY` 颜色仅用于子代理输出
3. **彩虹色**: 用于 ultrathink 关键词高亮
4. **选择背景**: `selectionBg` 用于终端鼠标选择高亮
5. **兼容性**: `themeColorToAnsi` 自动处理 256 色降级
