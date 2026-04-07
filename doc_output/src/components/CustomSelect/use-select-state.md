# use-select-state.ts

选择组件的状态管理 Hook，提供选择状态的基本管理功能。

## 基本信息

- **文件路径**: `src/components/CustomSelect/use-select-state.ts`
- **导出函数**: useSelectState
- **类型**: React Custom Hook

## 功能概述

useSelectState 提供了单选组件的状态管理能力，包括当前选中值、聚焦值、可见选项范围等。它内部使用 useSelectNavigation 处理导航逻辑。

## 核心内容

### Props 接口 (UseSelectStateProps<T>)

| 属性 | 类型 | 说明 |
|------|------|------|
| visibleOptionCount | number | 可见选项数量，默认 5 |
| options | OptionWithDescription<T>[] | 选项列表（必需） |
| defaultValue | T | 默认选中值 |
| onChange | (value: T) => void | 选择变更回调 |
| onCancel | () => void | 取消回调 |
| onFocus | (value: T) => void | 聚焦回调 |
| focusValue | T | 要聚焦的值 |

### 返回值 (SelectState<T>)

| 属性 | 类型 | 说明 |
|------|------|------|
| focusedValue | T \| undefined | 当前聚焦的选项值 |
| focusedIndex | number | 聚焦选项的 1-based 索引（0 表示无聚焦） |
| visibleFromIndex | number | 可见范围起始索引 |
| visibleToIndex | number | 可见范围结束索引 |
| value | T \| undefined | 当前选中的值 |
| options | OptionWithDescription<T>[] | 所有选项 |
| visibleOptions | Array<OptionWithDescription<T> & { index: number }> | 可见选项（带索引） |
| isInInput | boolean | 聚焦选项是否为输入类型 |
| focusNextOption | () => void | 聚焦下一个选项 |
| focusPreviousOption | () => void | 聚焦上一个选项 |
| focusNextPage | () => void | 聚焦下一页 |
| focusPreviousPage | () => void | 聚焦上一页 |
| focusOption | (value: T \| undefined) => void | 聚焦指定选项 |
| selectFocusedOption | () => void | 选中当前聚焦选项 |
| onChange | (value: T) => void | 选择变更回调 |
| onCancel | () => void | 取消回调 |

### 实现逻辑

1. 使用 `useState` 管理当前选中值 `value`
2. 使用 `useSelectNavigation` 管理导航状态
3. `selectFocusedOption` 将当前聚焦值设为选中值

## 设计要点

- 专注于单选状态管理，与多选状态分离
- 委托导航逻辑给 useSelectNavigation
- 使用 useCallback 缓存回调函数

## 与其他文件的关系

- **被导入**:
  - `select.tsx` - 主选择组件使用该 hook

- **导入依赖**:
  - `./use-select-navigation.js` - 导航逻辑 hook

## 备注

- 该 hook 主要用于单选场景，多选请使用 useMultiSelectState
- focusedIndex 返回的是 1-based 索引（从 1 开始），0 表示无聚焦
