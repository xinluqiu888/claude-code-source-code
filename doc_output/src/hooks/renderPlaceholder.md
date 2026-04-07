# renderPlaceholder.ts — 输入框占位符渲染器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/renderPlaceholder.ts`
- **类型**: TypeScript 工具模块
- **导出函数**: `renderPlaceholder`
- **依赖**: chalk（终端样式库）

## 功能概述

本模块负责渲染终端输入框的占位符（placeholder）文本，支持：
1. 普通占位符显示（灰色暗淡文本）
2. 光标位置指示器（反色显示）
3. 语音录制模式（仅显示光标，隐藏占位符文本）
4. 焦点状态感知

## 核心内容详解

### 接口定义

```typescript
type PlaceholderRendererProps = {
  placeholder?: string    // 占位符文本
  value: string          // 当前输入值
  showCursor?: boolean   // 是否显示光标
  focus?: boolean        // 输入框是否聚焦
  terminalFocus: boolean // 终端是否聚焦
  invert?: (text: string) => string  // 反色函数，默认 chalk.inverse
  hidePlaceholderText?: boolean  // 是否隐藏占位符文本（语音模式）
}
```

### 渲染逻辑

**1. 语音录制模式** (`hidePlaceholderText = true`)
- 仅显示反色光标（空格字符）
- 仅在输入框和终端都聚焦时显示
- 用于语音输入时减少视觉干扰

**2. 普通模式** (`hidePlaceholderText = false`)
- 占位符文本使用 `chalk.dim()` 变暗显示
- 光标显示为占位符第一个字符的反色
- 空占位符时显示反色空格

**3. 光标显示条件**
```typescript
showCursor && focus && terminalFocus
```
三个条件同时满足时才显示光标指示器

### 返回值

```typescript
{
  renderedPlaceholder: string | undefined  // 渲染后的占位符文本
  showPlaceholder: boolean                 // 是否应显示占位符
}
```

`showPlaceholder` 仅在 `value.length === 0 && Boolean(placeholder)` 时为 true

## 设计要点

### 1. 焦点状态分离

- `focus`: 输入框组件的聚焦状态
- `terminalFocus`: 整个终端窗口的聚焦状态
- 两者都需为 true 才显示光标，避免终端失焦时的视觉干扰

### 2. 样式处理

- 使用 chalk.inverse 实现反色光标效果
- 使用 chalk.dim 实现占位符文本变暗
- 支持自定义 invert 函数（便于测试或特殊主题）

### 3. 空值处理

- placeholder 为空字符串或 undefined 时返回 undefined
- 输入有值时 showPlaceholder 为 false

## 与其他文件的关系

- **chalk**: 第三方库，用于终端字符串样式
- **PromptInput 组件**: 调用此函数渲染输入框占位符
- **语音相关组件**: 设置 `hidePlaceholderText=true` 实现语音模式

## 注意事项

1. **终端兼容性**: chalk.dim 和 chalk.inverse 在大多数现代终端中正常工作
2. **性能**: 每次输入变化都可能触发渲染，但操作都是纯字符串处理，开销极小
3. **可访问性**: 暗淡的占位符文本可能在某些高对比度主题下不够明显
