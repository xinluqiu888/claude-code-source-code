# SelectMulti.tsx

多选选择组件，支持选择多个选项并提交。

## 基本信息

- **文件路径**: `src/components/CustomSelect/SelectMulti.tsx`
- **组件名称**: SelectMulti
- **类型**: React 函数组件 (使用 React Compiler)

## 功能概述

SelectMulti 组件提供多选功能，允许用户通过 Space 键或 Enter 键切换选项的选中状态。它支持输入类型选项、提交按钮、图片粘贴等高级功能。

## 核心内容

### Props 接口 (SelectMultiProps<T>)

| 属性 | 类型 | 说明 |
|------|------|------|
| isDisabled | boolean | 是否禁用，默认 false |
| visibleOptionCount | number | 可见选项数量，默认 5 |
| options | OptionWithDescription<T>[] | 选项列表 |
| defaultValue | T[] | 默认选中值数组 |
| onCancel | () => void | 取消回调（必需） |
| onChange | (values: T[]) => void | 选择变更回调 |
| onFocus | (value: T) => void | 聚焦回调 |
| focusValue | T | 聚焦值 |
| submitButtonText | string | 提交按钮文本（提供时显示提交按钮） |
| onSubmit | (values: T[]) => void | 提交回调 |
| hideIndexes | boolean | 隐藏序号，默认 false |
| onDownFromLastItem | () => void | 从最后一个选项向下回调 |
| onUpFromFirstItem | () => void | 从第一个选项向上回调 |
| initialFocusLast | boolean | 初始聚焦最后一个选项 |
| onOpenEditor | (...) => void | 打开外部编辑器回调 |
| onImagePaste | (...) => void | 图片粘贴回调 |
| pastedContents | Record<number, PastedContent> | 粘贴的内容 |
| onRemoveImage | (id: number) => void | 移除图片回调 |

### 交互逻辑

1. **无提交按钮时**:
   - Enter: 直接提交
   - Space: 切换选中状态

2. **有提交按钮时**:
   - Enter: 切换选中状态
   - 聚焦提交按钮 + Enter: 提交

### 选中状态显示

- 使用 `[✓]` 或 `[ ]` 标记显示选中状态
- 选中项显示为绿色（success 颜色）
- 聚焦项显示为 suggestion 颜色

## 设计要点

- 使用 `useMultiSelectState` 管理多选状态
- 支持输入类型选项，显示输入框供用户输入
- 自动计算序号宽度以实现对齐
- 显示向上/向下滚动箭头指示更多选项

## 与其他文件的关系

- **被导入**:
  - 各种需要多选功能的组件

- **导入依赖**:
  - `./use-multi-select-state.js` - 多选状态 hook
  - `./select-option.js` - 选项组件
  - `./select-input-option.js` - 输入选项组件
  - `../../ink.js` - Ink UI 库
  - `figures` - 终端图标库

## 备注

- `submitButtonText` 和 `onSubmit` 必须同时提供才能显示提交按钮
- 输入类型选项的选中标记显示方式与普通选项不同
- 支持通过 `hideIndexes` 隐藏序号，使界面更简洁
