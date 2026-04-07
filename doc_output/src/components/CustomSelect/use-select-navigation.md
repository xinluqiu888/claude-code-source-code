# use-select-navigation.ts

选择组件的导航状态管理 Hook，处理所有与导航相关的逻辑。

## 基本信息

- **文件路径**: `src/components/CustomSelect/use-select-navigation.ts`
- **导出函数**: useSelectNavigation
- **类型**: React Custom Hook

## 功能概述

useSelectNavigation 是选择组件的核心导航状态管理 hook。它使用 useReducer 实现复杂的导航逻辑，支持上下导航、分页导航、循环滚动、选项变化检测等功能。

## 核心内容

### State 类型

```typescript
type State<T> = {
  optionMap: OptionMap<T>      // 选项值到选项项的映射
  visibleOptionCount: number   // 可见选项数量
  focusedValue: T | undefined // 当前聚焦值
  visibleFromIndex: number     // 可见范围起始索引
  visibleToIndex: number       // 可见范围结束索引
}
```

### Action 类型

| Action | 说明 |
|--------|------|
| focus-next-option | 聚焦下一个选项 |
| focus-previous-option | 聚焦上一个选项 |
| focus-next-page | 聚焦下一页 |
| focus-previous-page | 聚焦上一页 |
| set-focus | 设置聚焦值 |
| reset | 重置状态 |

### Props 接口 (UseSelectNavigationProps<T>)

| 属性 | 类型 | 说明 |
|------|------|------|
| visibleOptionCount | number | 可见选项数量，默认 5 |
| options | OptionWithDescription<T>[] | 选项列表 |
| initialFocusValue | T | 初始聚焦值 |
| onFocus | (value: T) => void | 聚焦回调 |
| focusValue | T | 要聚焦的值（受控模式） |

### 返回值 (SelectNavigation<T>)

| 属性 | 类型 | 说明 |
|------|------|------|
| focusedValue | T \| undefined | 当前聚焦值 |
| focusedIndex | number | 聚焦索引（1-based） |
| visibleFromIndex | number | 可见起始索引 |
| visibleToIndex | number | 可见结束索引 |
| options | OptionWithDescription<T>[] | 所有选项 |
| visibleOptions | Array<...> | 可见选项列表 |
| isInInput | boolean | 是否在输入模式 |
| focusNextOption | () => void | 聚焦下一个 |
| focusPreviousOption | () => void | 聚焦上一个 |
| focusNextPage | () => void | 聚焦下一页 |
| focusPreviousPage | () => void | 聚焦上一页 |
| focusOption | (value?: T) => void | 聚焦指定选项 |

### Reducer 逻辑

1. **focus-next-option**:
   - 当前聚焦项有 next 时聚焦 next
   - 到达末尾时循环到第一个
   - 需要时滚动视口

2. **focus-previous-option**:
   - 当前聚焦项有 previous 时聚焦 previous
   - 到达开头时循环到最后一个
   - 需要时滚动视口

3. **focus-next-page**:
   - 向下移动 visibleOptionCount 个选项
   - 调整视口以显示新聚焦项

4. **focus-previous-page**:
   - 向上移动 visibleOptionCount 个选项
   - 调整视口以显示新聚焦项

5. **set-focus**:
   - 设置指定值为聚焦值
   - 如选项不在视口内，最小化滚动使其可见

6. **reset**:
   - 完全重置状态

## 设计要点

- 使用 `OptionMap` 类高效管理选项映射和链表关系
- 通过 `isDeepStrictEqual` 检测选项变化并自动重置
- 使用 ref 存储 onFocus 回调避免不必要的重渲染
- `validatedFocusedValue` 验证聚焦值是否存在于当前选项中

## 与其他文件的关系

- **被导入**:
  - `use-select-state.ts` - 单选状态 hook 使用该 hook
  - `use-multi-select-state.ts` - 多选状态 hook 使用该 hook

- **导入依赖**:
  - `./option-map.js` - 选项映射类
  - `util` 模块的 `isDeepStrictEqual`

## 备注

- focusedIndex 返回 1-based 索引（从 1 开始），0 表示无聚焦
- 选项变化时会自动重置状态，同时尽可能保留当前视口
- 支持通过 focusValue prop 实现受控模式的聚焦
