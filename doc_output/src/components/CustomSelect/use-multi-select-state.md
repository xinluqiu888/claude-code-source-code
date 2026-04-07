# use-multi-select-state.ts

多选组件的状态管理 Hook，提供多选状态的完整管理能力。

## 基本信息

- **文件路径**: `src/components/CustomSelect/use-multi-select-state.ts`
- **导出函数**: useMultiSelectState
- **类型**: React Custom Hook

## 功能概述

useMultiSelectState 是多选组件的核心状态管理 hook。它管理选中值数组、输入值映射、提交按钮聚焦状态等，并处理所有键盘输入逻辑。

## 核心内容

### Props 接口 (UseMultiSelectStateProps<T>)

| 属性 | 类型 | 说明 |
|------|------|------|
| isDisabled | boolean | 是否禁用，默认 false |
| visibleOptionCount | number | 可见选项数量，默认 5 |
| options | OptionWithDescription<T>[] | 选项列表（必需） |
| defaultValue | T[] | 默认选中值数组 |
| onChange | (values: T[]) => void | 选择变更回调 |
| onCancel | () => void | 取消回调（必需） |
| onFocus | (value: T) => void | 聚焦回调 |
| focusValue | T | 聚焦值 |
| submitButtonText | string | 提交按钮文本 |
| onSubmit | (values: T[]) => void | 提交回调 |
| onDownFromLastItem | () => void | 从最后一个选项向下回调 |
| onUpFromFirstItem | () => void | 从第一个选项向上回调 |
| initialFocusLast | boolean | 初始聚焦最后一个选项 |
| hideIndexes | boolean | 隐藏序号，默认 false |

### 返回值 (MultiSelectState<T>)

| 属性 | 类型 | 说明 |
|------|------|------|
| focusedValue | T \| undefined | 当前聚焦值 |
| visibleFromIndex | number | 可见范围起始索引 |
| visibleToIndex | number | 可见范围结束索引 |
| options | OptionWithDescription<T>[] | 所有选项 |
| visibleOptions | Array<...> | 可见选项列表 |
| isInInput | boolean | 是否在输入模式 |
| selectedValues | T[] | 选中的值数组 |
| inputValues | Map<T, string> | 输入选项的值映射 |
| isSubmitFocused | boolean | 提交按钮是否聚焦 |
| updateInputValue | (value: T, inputValue: string) => void | 更新输入值 |
| onCancel | () => void | 取消回调 |

### 状态管理

1. **selectedValues**: 使用 useState 管理选中值数组
   - 选项变化时通过 isDeepStrictEqual 检测并重置

2. **isSubmitFocused**: 提交按钮聚焦状态
   - 控制 Tab 导航是否聚焦提交按钮

3. **inputValues**: 输入选项的值映射
   - 初始化时从 options 中提取 initialValue
   - 通过 updateInputValue 更新

### 键盘输入处理

| 按键 | 行为 |
|------|------|
| Tab / Shift+Tab | 在选项和提交按钮之间导航 |
| Down / Ctrl+N / j | 下一个选项或聚焦提交按钮 |
| Up / Ctrl+P / k | 上一个选项或退出 |
| PageDown | 下一页 |
| PageUp | 上一页 |
| Enter / Space | 切换选中 / 提交 |
| Ctrl+Enter | 从输入字段提交 |
| 数字键 (1-9) | 直接切换对应索引选项的选中状态 |
| Escape | 取消 |

### 输入模式处理

- 在输入字段中时，只允许导航键（上下、Escape、Tab、Enter、Ctrl+N/P）
- 其他按键直接返回，让输入框处理

## 设计要点

- 使用 `useSelectNavigation` 处理导航逻辑
- 使用 `useRegisterOverlay` 注册为覆盖层
- 通过 `updateSelectedValues` 统一更新选中值和触发回调
- `updateInputValue` 同时更新内部状态和选项的 onChange

## 与其他文件的关系

- **被导入**:
  - `SelectMulti.tsx` - 多选组件使用该 hook

- **导入依赖**:
  - `./use-select-navigation.js` - 导航 hook
  - `../../context/overlayContext.js` - 覆盖层上下文
  - `util` 模块的 `isDeepStrictEqual`

## 备注

- 选项变化时会自动重置 selectedValues 到 defaultValue
- 这是为了解决异步加载数据后选项变化导致的选中状态不一致问题
- submitButtonText 和 onSubmit 同时存在时才会显示提交按钮
