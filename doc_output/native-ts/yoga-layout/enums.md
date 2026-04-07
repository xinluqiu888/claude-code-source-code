# yoga-layout/enums.ts — Yoga布局引擎枚举定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/native-ts/yoga-layout/enums.ts`
- **类型**: TypeScript 模块
- **作用**: 定义Yoga Flexbox布局引擎的枚举常量

## 功能概述

本模块从yoga-layout/src/generated/YGEnums.ts移植而来，定义了所有Yoga布局相关的枚举值。为了保持与代码库惯例一致，使用`const`对象而非TypeScript enum定义。

## 核心内容详解

### 对齐方式 (Align)

```typescript
export const Align = {
  Auto: 0,
  FlexStart: 1,
  Center: 2,
  FlexEnd: 3,
  Stretch: 4,
  Baseline: 5,
  SpaceBetween: 6,
  SpaceAround: 7,
  SpaceEvenly: 8,
} as const
```

### 盒模型 (BoxSizing)

```typescript
export const BoxSizing = {
  BorderBox: 0,   // 包含border和padding
  ContentBox: 1,  // 仅content
} as const
```

### 维度 (Dimension)

```typescript
export const Dimension = {
  Width: 0,
  Height: 1,
} as const
```

### 方向 (Direction)

```typescript
export const Direction = {
  Inherit: 0,
  LTR: 1,     // 从左到右
  RTL: 2,     // 从右到左
} as const
```

### 显示模式 (Display)

```typescript
export const Display = {
  Flex: 0,
  None: 1,        // 隐藏
  Contents: 2,    // 子元素提升到祖父节点
} as const
```

### 边 (Edge)

9边模型（支持物理边和逻辑边）：
```typescript
export const Edge = {
  Left: 0,
  Top: 1,
  Right: 2,
  Bottom: 3,
  Start: 4,       // 逻辑起始边（LTR=Left, RTL=Right）
  End: 5,         // 逻辑结束边
  Horizontal: 6,  // 水平边（同时设置Left和Right）
  Vertical: 7,    // 垂直边（同时设置Top和Bottom）
  All: 8,         // 所有边
} as const
```

### 错误修正 (Errata)

用于兼容性修正的位标志：
```typescript
export const Errata = {
  None: 0,
  StretchFlexBasis: 1,
  AbsolutePositionWithoutInsetsExcludesPadding: 2,
  AbsolutePercentAgainstInnerSize: 4,
  All: 2147483647,
  Classic: 2147483646,
} as const
```

### Flex方向 (FlexDirection)

```typescript
export const FlexDirection = {
  Column: 0,          // 垂直排列
  ColumnReverse: 1,   // 垂直反向
  Row: 2,             // 水平排列
  RowReverse: 3,      // 水平反向
} as const
```

### 间距 (Gutter)

```typescript
export const Gutter = {
  Column: 0,  // 列间距
  Row: 1,     // 行间距
  All: 2,     // 所有方向
} as const
```

### 对齐内容 (Justify)

主轴上的对齐方式：
```typescript
export const Justify = {
  FlexStart: 0,
  Center: 1,
  FlexEnd: 2,
  SpaceBetween: 3,
  SpaceAround: 4,
  SpaceEvenly: 5,
} as const
```

### 测量模式 (MeasureMode)

```typescript
export const MeasureMode = {
  Undefined: 0,   // 未定义
  Exactly: 1,     // 精确值
  AtMost: 2,      // 最大值
} as const
```

### 溢出 (Overflow)

```typescript
export const Overflow = {
  Visible: 0,
  Hidden: 1,
  Scroll: 2,
} as const
```

### 定位类型 (PositionType)

```typescript
export const PositionType = {
  Static: 0,    // 静态定位
  Relative: 1,  // 相对定位
  Absolute: 2,  // 绝对定位
} as const
```

### 单位 (Unit)

```typescript
export const Unit = {
  Undefined: 0, // 未定义
  Point: 1,     // 点（像素）
  Percent: 2,   // 百分比
  Auto: 3,      // 自动
} as const
```

### 换行 (Wrap)

```typescript
export const Wrap = {
  NoWrap: 0,        // 不换行
  Wrap: 1,          // 换行
  WrapReverse: 2,   // 反向换行
} as const
```

## 设计要点

1. **值与上游完全匹配**: 枚举值与Yoga原始实现完全一致，调用方无需更改
2. **使用const而非enum**: 符合代码库惯例，提供相同类型安全
3. **类型推导**: 每个const后导出对应类型，如`type Align = (typeof Align)[keyof typeof Align]`

## 与其他文件的关系

- **被 yoga-layout/index.ts 导入**: 布局引擎实现使用这些枚举
- **被 Ink 布局使用**: 终端UI组件使用Yoga进行flex布局

## 注意事项

1. **LTR假设**: Ink始终使用LTR方向，RTL未实现
2. **ContentBox未使用**: Ink不使用content-box模式
3. **AspectRatio未实现**: 未在Yoga中使用
